<template>
   <div class="card">
      <div class="card-header text-center" v-if="$slots.header">
         <slot name="header"></slot>
         <hr>
      </div>
      <div class="card-body">
         <div id="calendar"></div>
      </div>
   </div>
</template>

<script>
   // import * as d3 from "d3";
   import Card from './Card.vue'
   import Color2D from './color2d.vue'
   import draw_colormapVue from './draw_colormap.vue'
   // import Colormap from 'src/components/Dashboard/Views/Colormap.vue'
   
   export default {
      name: 'calendarview',
      props: ['yb', 'yd', 'sz'],
      components: {
         Card
      },
      mounted () {
         this.draw_calendar(d3v4,this.yb,this.yd,this.sz);
      },
      methods:{
         mouthPath(t0){
            let t1 = new Date(t0.getFullYear(), t0.getMonth() + 1, 0),
               d0 = t0.getDay(), w0 = d3.timeWeek.count(d3.timeYear(t0),t0),
               d1 = t1.getDay(), w1 = d3.timeWeek.count(d3.timeYear(t1),t1);
            return "M" + (w0 + 1) * cellSize + "," + d0 * cellSize
               + "H" + w0 * cellSize + "V" + 7 * cellSize
               + "H" + w1 * cellSize + "V" + (d1 + 1) * cellSize
               + "H" + (w1 + 1) * cellSize + "V" + 0
               + "H" + (w0 + 1) * cellSize + "Z";
         },
         draw_calendar(d3,yb,yd,size){
            const me = this;
            let [width,height,cellSize] = [960,136,17];

            yd = 1 + parseInt(yd);
            let format = d3.timeFormat("%Y-%m-%d");

            let svg = d3.select("#calendar").selectAll("svg")
               .data(d3.range(yb,yd))
               .enter().append("svg")
               .attr("width",width)
               .attr("height",height)
               .append("g")
               .attr("transform", "translate(" + ((width - cellSize * 53) / 2) + "," + (height - cellSize * 7 - 1) + ")");
            svg.append("text")
               .attr("transform", "translate(-6," + cellSize * 3.5 + ")rotate(-90)")
               .style("text-anchor","middle")
               .text(function(d){return d;});
            let rect = svg.selectAll(".day")
               .data(function(d) { return d3.timeDays(new Date(d, 0, 1), new Date(d + 1, 0, 1)); })
               .enter().append("rect")
               .attr("class", "day")
               .attr("width", cellSize)
               .attr("height", cellSize)
               .attr("x", function(d) { return d3.timeWeek.count(d3.timeYear(d),d) * cellSize; })
               .attr("y", function(d) { return d.getDay() * cellSize; })
               .datum(format);

            // svg.selectAll(".month")
            //    .data(function(d) { return d3.timeMonths(new Date(d, 0, 1), new Date(d + 1, 0, 1)); })
            //    .enter().append("path")
            //    .attr("class", "month")
            //    .attr("d", me.monthPath);

            rect.append("title")
               .text(function(d) { return d; });

            rect.style("fill",function (d) {
               let year_1 = new Date(d).getFullYear();
               let month_1 = new Date(d).getMonth()+1;
               let date_1 = new Date(d).getDate();
               let l1 = year_1+'-'+month_1+'-'+date_1;
               if (l1 in window.store){
                  let x = window.store[l1].position.x/512;
                  let y = window.store[l1].position.y/512;
                  let rgb = Color2D.getColor(x, y);
                  return "rgb("+rgb[0]+','+rgb[1]+','+rgb[2]+')';
               };
            });

            rect.on("click",function(d){
               let [year_1,month_1,date_1] = [new Date(d).getFullYear(),new Date(d).getMonth()+1,new Date(d).getDate()];
               let l1 = year_1+'/'+month_1+'/'+date_1;
               let dt1 = new Date(l1);
               let time2 = new Date((new Date(dt1)).setDate(dt1.getDate()+7));
               let time3 = new Date((new Date(dt1)).setDate(time2.getDate()-14));
               // console.log(time2,time3);
               
               for ( let i = 0; i < window.scatter_plot.children.length; i++){
                  let dt = new Date(window.scatter_plot.children[i].data["日期"]);
                  let time = dt.getTime();
                  if (time === dt1.getTime()){
                     window.scatter_plot.children[i].alpha = 1;
                  }
                  else if( time >= time3.getTime() && time <= time2.getTime() ){
                     window.scatter_plot.children[i].alpha = 0.1;
                  }  
                  else{
                     window.scatter_plot.children[i].alpha = 0;
                  }
               }
               me.parse2dad(me,l1,true);
            })
            .on("dblclick",function(d){
               for ( let i = 0; i < window.scatter_plot.children.length; i++){
                  window.scatter_plot.children[i].alpha = 1;
               }
               me.parse2dad(me,"nodisplay",false);
            });
         },
         changefill(d3){
            let svg = d3.select("#calendar").selectAll("svg");
            let rect = svg.selectAll(".day");
            rect.style("fill",function (d) {
               let year_1 = new Date(d).getFullYear();
               let month_1 = new Date(d).getMonth()+1;
               let date_1 = new Date(d).getDate();
               let l1 = year_1+'-'+month_1+'-'+date_1;
               if (l1 in window.store){
                  let x = window.store[l1].position.x/512;
                  let y = window.store[l1].position.y/512;
                  let rgb = Color2D.getColor(x, y);
                  return "rgb("+rgb[0]+','+rgb[1]+','+rgb[2]+')';
               };
            });
         },
         parse2dad(me,date,display){
            me.$emit('father',date,display);
         },
      },
   };
</script>
<style>
.day {
   fill: #fff;
   stroke: #ccc;
}

.month {
   fill: none;
   stroke: #000;
   stroke-width: 2px;
}

#calendar{
   text-align:center ;
}
.Card{
   z-index: 9999;
}
</style>