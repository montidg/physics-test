<style>
    svg {
        border: solid black 2px;
    } 
</style>

<script>
    let rect = {
        points: [
            [13.820394,173.18048],
            [12.968946,53.56121],
            [25.195312,53.6505],
            [76.688089,148.57609],
            [40.114529,116.49192],
            [32.439234,156.44847], 
            [86.079789,195.27127],
            [108.40258,116.98358],[61.572425,86.262567],[93.871155,31.254717],[134.19208,47.428505],[155.85331,170.43225],[106.78994,266.45433],[24.783022,262.22076],[79.786008,235.52481],[33.905287,209.06263],[5.0651629,204.20621]
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
        _right: 0
    };

    let stage = {
        w: 1
    };

    let floor = {
        y: 900
    };

    function ground(obj) {
        if (obj.y > floor.y - obj._bottom) {
            obj._yVel *= 0;
            obj.y = floor.y - obj._bottom;
        }

        return obj;
    }

    function vel(obj) {
        obj._yVel += 0.04;
        
        obj._rVel *= 0.99;
        obj._yVel *= 0.995;
        obj._xVel *= 0.995;

        obj.x += obj._xVel;
        obj.y += obj._yVel;
        obj.rotation += obj._rVel;

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

        obj._right = bData[bData.length - 1];
        obj._left = bData[0];

        return obj;
    }

    function main() {
        rect = vel(rect);
        rect = length(rect);
        rect = ground(rect);
    }

    function clickFunc(e) {
        let off = (e.offsetX / (stage.w / 1920) - rect.x - (rect._left) / 2) / -20;
        console.log(rect._strLen);
        rect._yVel += -5;
        rect._xVel += off;
    }

    function downFunc(e) {

    }

    function upFunc(e) {

    }

    rect = partition(rect);
    setInterval(main,0);
</script>

<div width="80vw" height="45vw" bind:clientWidth={stage.w} >
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
