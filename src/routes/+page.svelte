<style>
    svg {
        border: solid black 2px;
    } 
</style>

<script>
    let rect = {
        points: [
            [0,0],
            [0,100],
            [100,200],
            [100,0]
        ],
        x: 100,
        y: 100,
        rotation: 25,

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
        _bottomN: 0,

        _center: [0,0],
        _centerT: [0,0]
    };

    let stage = {
        w: 1,
        x: 0,
        y: 0,
        elem: null
    };

    let floor = {
        y: 900
    };

    function ground(obj) {
        if (obj.y > floor.y - obj._bottom) {
            obj._yVel *= -1;
            obj.y = floor.y - obj._bottom;

            if (Math.abs(obj._bottom - obj._bottomN) > 2) {
                obj._rVel += (obj._centerT[0] - obj._bottomX) / 100
            } else {
                obj._rVel = 0;
            }
            
        }

        return obj;
    }

    function vel(obj) {
        obj._yVel += 0.04;
        
        obj._rVel *= 0.98;
        obj._yVel *= 0.995;
        obj._xVel *= 0.995;

        obj.x += obj._xVel;
        obj.y += obj._yVel;
        obj.rotation += obj._rVel;
        
        let off = rotate(obj._centerT[0], obj._centerT[1], obj._rVel / 180 * Math.PI);

        obj.x -= obj._centerT[0] - off[0];
        obj.y -= obj._centerT[1] - off[1];

        return obj;
    }

    function partition(obj) {
        obj._path = 'M ' + rect.points.map(x => x.join(' ')).join(' L ') + ' Z';   
        
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

    function rotate(x,y,d) {
        return [x * Math.cos(d) + y * Math.sin(d), y * Math.cos(d) - x * Math.sin(d)];
    }

    function length(obj) {
        obj._radians = obj.rotation / 180 * Math.PI;

        obj._trianglesT = obj._triangles.map(tri => {
            tri.data = tri.data.map((arr) => rotate(arr[0],arr[1],obj._radians));
            tri.center = rotate(tri.center[0],tri.center[1],-obj._radians);
            return tri;
        })

        obj._pointsT = obj.points.map(point => {
            return rotate(point[0],point[1],-obj._radians)
        })

        let bData = obj._pointsT.map(tri => {
            return tri[1];
        }).sort((a,b) => a - b);

        let bDataH = obj._pointsT.map(tri => {
            return tri[0];
        }).sort((a,b) => a - b);

        obj._bottom = bData[bData.length - 1];
        obj._top = bData[0];

        obj._right = bDataH[bDataH.length - 1];
        obj._left = bDataH[0];

        obj._bottomX = obj._pointsT.sort((a,b) => a[1] - b[1])[bData.length - 1][0];
        obj._bottomN = bData[bData.length - 2];

        obj._centerT = rotate(obj._center[0], obj._center[1], -obj._radians);

        return obj;
    }

    function main() {
        rect = length(rect);
        rect = vel(rect);
        rect = ground(rect);

        if (rect.x > 1900){ 
            rect.x = 0;
        }

        if (rect.x < -50){ 
            rect.x = 1900;
        }
    }

    function clickFunc(e) {
        let x = (e.clientX - stage.elem.offsetLeft)  / (stage.w / 1920);
        let off = (x - rect.x - rect._center[0]) / -20;
        rect._yVel += -5;
        rect._xVel += off;
        rect._rVel += off * 2;
    }

    function downFunc(e) {

    }

    function upFunc(e) {

    }

    rect = partition(rect);
    setInterval(main,0);
</script>

<div width="80vw" height="45vw" bind:clientWidth={stage.w} bind:this={stage.elem} >
    <svg width="80vw" height="45vw" viewBox="0 0 1920 1080">
        <path fill="rgb(0,45,90)" d="M 0 900 h 1920 v 180 h -1920 Z"/>
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <path 
            on:click={clickFunc}
            on:mousedown={downFunc}
            on:mouseup={upFunc}
            d={rect._path}
            transform='
                translate(
                    {rect.x},
                    {rect.y}
                ) 
                rotate(
                    {rect.rotation} 
                )
            '>
        </path>
    </svg>
</div>
