<style>
    svg {
        border: solid black 2px;
    } 
</style>

<script>
    let rect = {
        width: 100,
        height: 100,
        x: 100,
        y: 100,
        rotation: 25,

        _radians: 0,

        _xVel: 0,
        _yVel: 0,
        _rVel: 0,

        _len: 0,
        _leftLen: 0,
        _rightLen: 0,
        _minLen: 0,
        _strLen: 0,

        _bottom: 0
    };

    let stage = {
        w: 1
    };

    let floor = {
        y: 900
    };

    function ground(obj, surface) {
        if (surface) {
            obj._yVel *= -1.5;
            obj.y = floor.y - obj._bottom + obj.y;
        }     

        let factor = ((obj._leftLen > obj._rightLen) ? 1 : -1);
        factor *= (obj._strLen  / obj.height);
        factor *= (surface) ? 0.05 : 0.02;

        obj._rVel += factor;

        return obj;
    }

    function collide(obj) {

        obj._bottom = obj.y + (obj._len * obj.height);

        obj = ground(obj, (obj._bottom >= floor.y) );

        return obj;
    }

    function length(obj) {
        obj._radians = obj.rotation/180 * Math.PI;

        obj._len = Math.max(
            Math.abs(Math.sin(obj._radians + Math.PI / 4)), 
            Math.abs(Math.sin(obj._radians - Math.PI / 4))
        ) / Math.sqrt(2);

        obj._leftLen = Math.abs(Math.sin(obj._radians + Math.PI / 4));
        obj._rightLen = Math.abs(Math.sin(obj._radians - Math.PI / 4));

        obj._minLen= Math.min(
            obj._leftLen, 
            obj._rightLen
        ) / Math.sqrt(2);

        obj._strLen = (obj._len * obj.height) + obj._minLen * obj.height;

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

    function main() {
        rect = vel(rect);
        rect = length(rect);
        rect = collide(rect);        
    }

    function clickFunc(e) {
        let off = (e.offsetX / (stage.w / 1920) - rect.x - (rect._len)) / -20;
        console.log(rect._strLen);
        rect._yVel += -5;
        rect._rVel += off;
        rect._xVel += off;
    }

    function downFunc(e) {

    }

    function upFunc(e) {

    }

    setInterval(main,0);
</script>

<div width="80vw" height="45vw" bind:clientWidth={stage.w} >
    <svg width="80vw" height="45vw" viewBox="0 0 1920 1080">
        <path fill="rgb(0,45,90)" d="M 0 900 h 1920 v 180 h -1920 Z"/>
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <rect 
            on:click={clickFunc}
            on:mousedown={downFunc}
            on:mouseup={upFunc}
            width={rect.width} 
            height={rect.height} 
            transform='
                translate(
                    {rect.x - rect.width/2},
                    {rect.y - rect.height/2}
                ) 
                rotate(
                    {rect.rotation} 
                    {rect.width / 2} 
                    {rect.height / 2}
                )
            '>
        </rect>
    </svg>
</div>
