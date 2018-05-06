<script>
var Color2D = {};

Color2D.imgFolder = "./static";

/*
 * dimensions of the colormap image
 */
Color2D.dimensions = {
    width: 512,
    height: 512
};

/*
 * Available 2D colormaps - reference the png file
 */
Color2D.colormaps = {
    BREMM:      Color2D.imgFolder + "/map.png",
   //  SCHUMANN:   Color2D.imgFolder + "schumann.png",
   //  STEIGER:    Color2D.imgFolder + "steiger.png",
   //  TEULING2:   Color2D.imgFolder + "teulingfig2.png",
   //  ZIEGLER:    Color2D.imgFolder + "ziegler.png"
};

/*
 * Active colormap - if you want to set another one, call Color2D.setColormap(c)
 */
Color2D.colormap = Color2D.colormaps.BREMM; // standard colormap

Color2D.context = null;

/*
 * initializes the 2D colormap
 */
Color2D.init = function(callback) {
    // create invisible canvas element in dom tree
    let cm = document.getElementById("cm");
    var canvas = document.createElement("canvas");
    canvas.id = "colormap";
    canvas.width = String(Color2D.dimensions.width);
    canvas.height = String(Color2D.dimensions.height);
    canvas.style = "display:none";
    cm.appendChild(canvas);

    // create canvas context for later color reading
    Color2D.context = canvas.getContext("2d");
    // draw colormap image
    var imgObj = new Image();
    imgObj.onload = function() {
        Color2D.context.drawImage(imgObj, 0, 0);
        callback();
    };
    imgObj.src = Color2D.colormap;
};

/*
 * data ranges = min and max values of x and y dimensions
 */
Color2D.ranges = {
    x: [0, 1],
    y: [0, 1]
}

/*
 * computes the scaled X value
 */
Color2D.getScaledX = function(x) {
    var val = ((x+1) - (Color2D.ranges.x[0]+1)) / ((Color2D.ranges.x[1]+1) - (Color2D.ranges.x[0]+1));
    return (val * (Color2D.dimensions.width-1));
};

/*
 * computes the scaled Y value
 */
Color2D.getScaledY = function(y) {
    var val = ((y+1) - (Color2D.ranges.y[0]+1)) / ((Color2D.ranges.y[1]+1) - (Color2D.ranges.y[0]+1));
    return (val * (Color2D.dimensions.height-1));
};

/*
 * set a new 2D colormap
 */
Color2D.setColormap = function(colormap, callback) {
    Color2D.colormap = colormap;
    // reset canvas
    var element = document.getElementById("colormap");
    if (element !== null) {
        element.outerHTML = "";
        delete document.getElementById("colormap");
    }
    // init new canvas
    Color2D.init(callback);
};

/*
 * get the color for a x and y position in space
 */
Color2D.getColor = function(x, y) {
    //var color = Color2D.context.getImageData(x, y, 1, 1); // rgba [0, 255] // not scaled
    var color = Color2D.context.getImageData(Color2D.getScaledX(x), Color2D.getScaledY(y), 1, 1); // rgba [0, 255]
    var r = color.data[0];
    var g = color.data[1];
    var b = color.data[2];
    return [r, g, b];
};

export default Color2D; 
</script>