<template>
   <div class="card">
      <div class="card-header text-center" v-if="$slots.header">
         <slot name="header"></slot>
         <hr>
      </div>
      <div class="card-body text-center" v-if="$slots.body">
         <slot name="body"></slot>
      </div>
      <div class="card-footer text-center" v-if="$slots.footer">
         <slot name="footer"></slot>
      </div>
   </div>
</template>

<script>
   // import * as d3 from "d3";
   import Card from './Card.vue'
   import Color2D from './color2d.vue'
   import Calendar from 'src/components/UIComponents/Cards/calendarview.vue'
   import calendarviewVue from './calendarview.vue';
   export default {
      name: 'colormap',
      components: {
         Card
      },
      methods:{
         getColor(x,y){
            var rgb = Color2D.getColor(x, y);
            return rgb[0] * 0x10000 + rgb[1] * 0x100 + rgb[2];            
         },
         init(){
            let getColor = (x,y)=>{
               let rgb = Color2D.getColor(x,y);
               return rgb[0] * 0x10000 + rgb[1] * 0x100 + rgb[2]; 
            }
            window.sp = {};
            let type='webGL'
            if(!PIXI.utils.isWebGLSupported()){
               type = "canvas"
            }
            PIXI.utils.sayHello(type)

            //Create a Pixi Application
            let cm = document.getElementById("cm");
            var app = new PIXI.Application({
               width: 512, height: 512,
               backgroundColor: 0xFFFFFF,
               antialias: true,
               view: document.getElementById('pixiCanvas'),
            });
            window.app = app;

            cm.appendChild(app.view);

            window.scatter_plot = new PIXI.Container();
            var bg2 = PIXI.Sprite.fromImage('static/map.png');
            bg2.alpha = 0.7;
            bg2.buttonMode = true;
            window.bg2 = bg2;
            app.stage.addChild(bg2);
            var rect = new PIXI.Graphics();
            rect.lineStyle(2, 0x000000, 1);
            rect.drawRect(0, 0, 512, 512);
            app.stage.addChild(rect);
            app.stage.addChild(scatter_plot);

            d3.csv('static/Plastics_and_Chemicals_0.csv', function(data) {
               window.csv_data = data;
               let xmin = data[0].latent_dim1, xmax = data[0].latent_dim1,
                  ymin = data[0].latent_dim2, ymax = data[0].latent_dim2;
               for (let d of data) {
                  xmax = Math.max(xmax, d.latent_dim1);
                  xmin = Math.min(xmin, d.latent_dim1);

                  ymax = Math.max(ymax, d.latent_dim2);
                  ymin = Math.min(ymin, d.latent_dim2);
               }
               let xrange = xmax - xmin;
               let yrange = ymax - ymin;
               window.sp.xrange = xrange;
               window.sp.yrange = yrange;
               window.sp.xmin = xmin;
               window.sp.ymin = ymin;

               var points = new PIXI.Graphics();
               window.store = {};
               window.size = 0;
               points.lineStyle(0);
               window.scatter_plot.data = data;
               //points.beginFill(0xFF, 1);
               for (let d of data) {
                  // draw a circle
                  let x = (d.latent_dim1 - xmin) / xrange;
                  let y = (d.latent_dim2 - ymin) / yrange;
                  d.x = x;
                  d.y = y;
                  d.position = [x * 512, y * 512];
                  var p = new PIXI.Graphics();
                  p.data = d;
                  let year = new Date(p.data["日期"]).getFullYear();
                  let month = new Date(p.data["日期"]).getMonth()+1;
                  let day = new Date(p.data["日期"]).getDate();
                  let l2 = year+'-'+month+'-'+day;
                  window.size++;
                  p.lineStyle(1);
                  p.beginFill(getColor(x, y), 1);
                  p.drawCircle(0, 0, 3);
                  p.endFill();
                  p.pos = [x * 512, y * 512];
                  p.position.x = x * 512;
                  p.position.y = y * 512;
                  window.scatter_plot.addChild(p);
                  // scatter_plot.points.push(p);
                  window.store[l2] = p ; 
               }   
               // window.store.scatter_plot = scatter_plot;
            });
            // console.log(scatter_plot.children);
            let zoom = this.zoom;
            d3v4.select('#pixiCanvas').call(d3v4.zoom().scaleExtent([0.5, 20]).on("zoom",zoom));
         },
         zoom(){
            let transform = d3v4.event.transform;
            for( let d in window.store )
            {
               let p = window.store[d];
               let pos = transform.apply(p['pos']);
               p.position.x = pos[0];
               p.position.y = pos[1];
            }
         },
         remap(){
            for( let d in window.store ){
               let p = window.store[d];
               let [x,y] = [p.position.x/512,p.position.y/512];
               if (x >= 0 && x <= 1 && y >= 0 && y <= 1){
                  p.clear();
                  p.color = this.getColor(x,y);
                  p.lineStyle(1);
                  p.beginFill(p.color,1);
                  p.drawCircle(0,0,3);
                  p.endFill();
               }
            }
         },
         createDragAndDropFor(target){
            target.interactive = true;
            target.on("mousedown",function(e){
               this.drag = true;
            });
            target.on("mouseup",function(e){
               this.drag = false;
            });
            target.on("mousemove",function(e){
               if(this.drag){
                  this.position.x += e.data.originalEvent.movementX;
                  this.position.y += e.data.originalEvent.movementY;
               }
            });
         },
         keyborad(keyCode){
            let key = {};
            key.code    = keyCode;
            key.isDown  = false;
            key.isUp    = true;
            key.press   = undefined;
            key.release = undefined;
            // the down handler 
            key.downHandler = event =>{
               if (event.keyCode === key.code){
                  if (key.isUp && key.press) key.press();
                  key.isDown = true;
                  key.isUp   = false;
               }
               event.preventDefault();
            };

          	key.upHandler = event => {
               if (event.keyCode === key.code) {
                  if (key.isDown && key.release) key.release();
                  key.isDown = false;
                  key.isUp = true;
               }
               event.preventDefault();
            };  

            window.addEventListener(
               "keydown", key.downHandler.bind(key), false
            );
            window.addEventListener(
               "keyup", key.upHandler.bind(key), false
            );

            return key;
         },
      },
      async mounted () {
         let initial = this.init();
         Color2D.setColormap(Color2D.colormaps.BREMM,function(){
            initial;
         })
      }
   };
</script>
<style>
#colormap{
   text-align:center ;
}
</style>