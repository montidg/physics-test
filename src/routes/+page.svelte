<style>
    svg {
        border: solid black 2px;
    } 
    h1 {
        font-family: sans-serif;
        text-align: center;
    }
</style>

<script>
    function initObj({points, x, y, rotation}) {
        let obj = {
            points,x,y,rotation,

            _radians: 0,

            _xVel: 0,
            _yVel: 0,
            _rVel: 0,

            _path: '',

            _triangles: [],
            _trianglesT: [],

            _pointsT: [],

            _bottom: 0,
            _top: 0,
            _left: 0,
            _right: 0,

            _bottomX: 0,
            _topX: 0,
            _leftX: 0,
            _rightX: 0,

            _center: [0,0],
            _centerT: [0,0],

            _oldX: 0,
            _oldY: 0,

            _collide: [],
            _signature: [0,0]
        };

        return obj;
    }

    // credit to https://stackoverflow.com/questions/2778240/detection-of-triangle-collision-in-2d-space
    function cross(points, triangle) {
        var pa = points.data[0];
        var pb = points.data[1];
        var pc = points.data[2];

        var p0 = triangle.data[0];
        var p1 = triangle.data[1];
        var p2 = triangle.data[2];

        var dXa = pa[0] - p2[0];
        var dYa = pa[1] - p2[1];
        var dXb = pb[0] - p2[0];
        var dYb = pb[1] - p2[1];
        var dXc = pc[0] - p2[0];
        var dYc = pc[1] - p2[1];
        var dX21 = p2[0] - p1[0];
        var dY12 = p1[1] - p2[1];
        var D = dY12 * (p0[0] - p2[0]) + dX21 * (p0[1] - p2[1]);
        var sa = dY12 * dXa + dX21 * dYa;
        var sb = dY12 * dXb + dX21 * dYb;
        var sc = dY12 * dXc + dX21 * dYc;
        var ta = (p2[1] - p0[1]) * dXa + (p0[0] - p2[0]) * dYa;
        var tb = (p2[1] - p0[1]) * dXb + (p0[0] - p2[0]) * dYb;
        var tc = (p2[1] - p0[1]) * dXc + (p0[0] - p2[0]) * dYc;
        if (D < 0) return ((sa >= 0 && sb >= 0 && sc >= 0) ||
                            (ta >= 0 && tb >= 0 && tc >= 0) ||
                            (sa+ta <= D && sb+tb <= D && sc+tc <= D));
        return ((sa <= 0 && sb <= 0 && sc <= 0) ||
                (ta <= 0 && tb <= 0 && tc <= 0) ||
                (sa+ta >= D && sb+tb >= D && sc+tc >= D));
    }

    function collide(j, area) {
        let obj = area[j];
        obj._collide = [];

        for (var i = 0; i < area.length; i++) {
            let compar = area[i];
            if (
                compar._bottom < obj._top || 
                compar._top > obj._bottom || 
                compar._right < obj._left || 
                compar._left > obj._right ||
                i == j
            ) continue; 

            let triA = obj._trianglesT;
            let triB = compar._trianglesT;

            let found = false;
            for (var x = 0; x < triA.length; x++) {
                for (var y = 0; y < triB.length; y++) {
                    if (!(cross(triA[x],triB[y]) || cross(triB[y],triA[x]))) {
                        obj._collide.push([i]);
                        found = true;
                    }
                }
            }
        }

        return obj;
    }
    
    function ground(obj, dir,a,b) {
        obj._signature = [0,0];
        if (obj._collide.length > 0 && (a != obj._signature[0] || b != obj._signature[1])) {
            if (a != 0) {
                obj._signature[0] = -a;
            }

            if (b != 0) {
                obj._signature[1] = -b;
            }
            
            if (dir == 1) {
                obj.y = obj._oldY - obj._yVel * 3;
                obj._yVel = 0;             

                if (obj._signature[1] == -1) {
                    obj._rVel += (obj._centerT[0] - obj._bottomX + obj.x) / 2000
                } else {
                    obj._rVel = 0
                }
                
            } else {
                
                obj.x = obj._oldX - obj._xVel * 3;
                obj._xVel = 0;
            }
              
            
        } else {
            if (dir == 1) {
                obj._oldX = obj.x;
            } else {
                obj._oldY = obj.y;
            }
        }
        

        return obj;
    }

    function vel(obj) {
        if (obj._signature[1] == 0) {
            obj._yVel += 0.04;
        }
        
        obj._rVel *= 0.98;
        obj._yVel *= 0.995;
        obj._xVel *= 0.995;

        obj.y += obj._yVel;
        obj = collide(0,area);
        obj = ground(obj,1,Math.sign(obj._xVel),Math.sign(obj._yVel));

        obj.x += obj._xVel;
        obj = collide(0,area);
        obj = ground(obj,0,Math.sign(obj._xVel),Math.sign(obj._yVel));

        obj.rotation += obj._rVel;

        return obj;
    }

    function partition(obj) {
        obj._path = 'M ' + obj.points.map(x => x.join(' ')).join(' L ') + ' Z';   
        
        for (var i = 2; i < obj.points.length; i++) {

            let pointData = [obj.points[0], obj.points[i-1], obj.points[i]];

            let xd = pointData[0][0] - pointData[1][0];
            let yd = pointData[0][1] - pointData[1][1];

            let xdl = pointData[2][0] - pointData[1][0];
            let ydl = pointData[2][1] - pointData[1][1];

            let l = Math.sqrt(xd * xd + yd * yd);
            let w = Math.abs(xd * ydl - yd * xdl)/Math.sqrt(xd * xd + yd * yd);

            let center = [
                (pointData[0][0] + pointData[1][0] + pointData[2][0]) / 3,
                (pointData[0][1] + pointData[1][1] + pointData[2][1]) / 3                
            ];

            obj._triangles[i-2] = {
                data: pointData,
                mass: (l * w)/2,
                center: center
            };
        }

        let avg = [0,0];
        let totalMass = 0;
        for (var i = 0; i < obj._triangles.length; i++) {
            let mass = obj._triangles[i].mass;
            totalMass += mass;
            avg[0] += obj._triangles[i].center[0] * mass;
            avg[1] += obj._triangles[i].center[1] * mass;
        }

        avg[0] /= totalMass;
        avg[1] /= totalMass;

        obj._center = avg;

        return obj;
    }

    function rotate(x,y,d,c) {
        x -= c[0];
        y -= c[1];
        return [x * Math.cos(d) + y * Math.sin(d) + c[0], y * Math.cos(d) - x * Math.sin(d) + c[1]];
    }

    function length(obj) {
        obj._radians = obj.rotation / 180 * Math.PI;

        obj._trianglesT = obj._triangles.map(tri => {
            let newTri = {};

            newTri.data = [];

            for (var i = 0; i < tri.data.length; i++) {
                let arr = tri.data[i];
                let out = rotate(arr[0],arr[1],-obj._radians,obj._center);
                let out2 = [out[0]+obj.x,out[1]+obj.y];
                newTri.data.push(out2);
            };

            newTri.center = rotate(tri.center[0],tri.center[1],-obj._radians,obj._center);
            return newTri;
        })

        obj._pointsT = obj.points.map(point => {
            return rotate(point[0],point[1],-obj._radians,obj._center)
        })

        let bData = obj._pointsT.map(tri => {
            return tri;
        }).sort((a,b) => a[1] - b[1]);

        let bDataH = obj._pointsT.map(tri => {
            return tri;
        }).sort((a,b) => a[0] - b[0]);

        obj._bottom = bData[bData.length - 1][1] + obj.y;
        obj._bottomX = bData[bData.length - 1][0] + obj.x;
        obj._top = bData[0][1] + obj.y;
        obj._topX = bData[0][0] + obj.x;

        obj._right = bDataH[bDataH.length - 1][0] + obj.x;
        obj._rightY = bDataH[bDataH.length - 1][1] + obj.y;
        obj._left = bDataH[0][0] + obj.x;
        obj._leftY = bDataH[0][1] + obj.y;

        obj._centerT = obj._center;

        return obj;
    }

    function main() {
        for (var i = 0; i < area.length; i++) {
            area[i] = length(area[i]);
            area[i].index = i;
        }

        area[0] = vel(area[0]);

        stage.x += 1920 / 2;
        stage.y += 1300 / 2;
        stage.x = stage.x * 0.8 + (area[0].x) * 0.2;
        stage.y = stage.y * 0.8 + (area[0].y) * 0.2;
        stage.x -= 1920 / 2;
        stage.y -= 1300 / 2;
    }

    function clickFunc(e, i) {
        if (i != 0) return;
        let x = (e.clientX - stage.elem.offsetLeft)  / (stage.w / 1920) + stage.x;
        let off = (x - area[i].x - area[i]._center[0]) / -20;
       
        if (area[i]._collide.length == 0) {
            area[i]._rVel += off * 2;
            area[i]._yVel += -5;
            area[i]._xVel += off;
        }
        
    }

    function downFunc(e) {

    }

    function upFunc(e) {

    }

    let area = [
        initObj({
            points: [
                [0,0],
                [0,100],
                [100,200],
                [100,0]
            ],
            x: 100,
            y: 100,
            rotation: 25,
        }),
        initObj({
            points: [
                [1920,900],
                [1920,1080],
                [0,1080],
                [0,900],
                [1920/2,700],
            ],
            x: 0,
            y: 0,
            rotation: 0,
        }),
        initObj({
            points: [
                [0,-900],
                [0,-1080],
                [1920,-1080],
                [1920,-900]
            ],
            x: 0,
            y: 0,
            rotation: 0,
        }),
        initObj({
            points: [
                [-500,1080],
                [0,1080],
                [0,-1080],
                [-500,-1080]
            ],
            x: 0,
            y: 0,
            rotation: 0,
        }),
        initObj({
            points: [
                [1920+500,1080],
                [1920,1080],
                [1920,-1080],
                [1920+500,-1080]
            ],
            x: 0,
            y: 0,
            rotation: 0,
        }),
    ];

    area = area.map(x => partition(x));

    let stage = {
        w: 1,
        x: 0,
        y: 0,
        elem: null
    };
    setInterval(main,0);
</script>

<h1>
    Click the shape in the center to bounce it.
</h1>
<div width="60vw" height="33.75vw" bind:clientWidth={stage.w} bind:this={stage.elem}>
    <svg width="60vw" height="33.75vw" viewBox="0 0 1920 1080">
        <g transform='translate({-stage.x},{-stage.y})'>
            {#each area as obj}
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <path 
                    on:click={(e) => clickFunc(e,obj.index)}
                    d={obj._path}
                    transform='
                        translate(
                            {obj.x+(obj._center[0])},
                            {obj.y+(obj._center[1])}
                        ) 
                        rotate(
                            {obj.rotation}
                        ),
                        translate(
                            {-(obj._center[0])},
                            {-(obj._center[1])}
                        ) 
                    '>
                </path>
            {/each}
        </g>
        
    </svg>
</div>
