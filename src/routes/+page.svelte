<style>
    svg {
        border: solid black 2px;
    } 
</style>

<script>
    let rect = {
        w: 100,
        h: 100,
        x: 100,
        y: 100,
        r: 25,
        xv: 0,
        yv: 0,
        rv: 0
    };

    let floor = {
        y: 900
    };

    let lengthStraight = 0;

    function main() {
        rect.yv += 0.1;
        rect.y += rect.yv;
        rect.r += rect.rv;

        let length = Math.max(
            Math.abs(Math.sin(rect.r/180 * Math.PI + Math.PI / 4)), 
            Math.abs(Math.sin(rect.r/180 * Math.PI - Math.PI / 4))
        ) / Math.sqrt(2);

        let lengthA = Math.abs(Math.sin(rect.r/180 * Math.PI + Math.PI / 4));
        let lengthB = Math.abs(Math.sin(rect.r/180 * Math.PI - Math.PI / 4));

        let lengthMin = Math.min(
            lengthA, 
            lengthB
        ) / Math.sqrt(2);

        let bottom = rect.y + (length * rect.h);

        if (bottom >= floor.y) {
            rect.yv = 0;
            rect.y = floor.y - bottom + rect.y;

            lengthStraight = (length * rect.h) + lengthMin * rect.h;

            if (lengthA > lengthB) {
                rect.rv -= (1 - (lengthStraight / rect.h))
            } else {
                rect.rv += (1 - (lengthStraight / rect.h));
            }
        }
    }

    setInterval(main,0);
</script>

<svg width="80vw" height="auto" viewBox="0 0 1920 1080">
    <path fill="rgb(0,45,90)" d="M 0 900 h 1920 v 180 h -1920 Z"/>
    <rect width={rect.w} height={rect.h} transform='translate({rect.x - rect.w/2},{rect.y - rect.h/2}) rotate({rect.r} {rect.w / 2} {rect.h / 2})'></rect>
</svg>