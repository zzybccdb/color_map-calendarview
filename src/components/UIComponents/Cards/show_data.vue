<template>
   <div class="card">
      <div class="card-header text-center" v-if="$slots.header">
         <slot name="header"></slot>
         <hr>
      </div>
      <div class="card-body">
         <div id="showdata"></div>
      </div>
   </div>
</template>
<script>
  import Card from './Card.vue'
  export default {
   name: 'stats-card',
   props: ['date'],
   components: {
      Card
   },
   mounted () {
      this.initial(d3v4,this.date);
   },
   methods:{
      initial(d3,cl_date){
         const me = this;
         const a = d3.rgb(200, 242, 116);
         const b = d3.rgb(242, 79, 79);
         const c = d3.rgb(255, 255, 255);
         let compute = d3.interpolate(a,b);  
         let linear = d3.scaleLinear()
                .domain([-0.1,0,0.1])  
                .range([a,c,b]); 

         d3.selectAll("#showdata > *").remove();

         const [width,height,cellSize] = [960,20,20];

         let dt = new Date(cl_date);
         let dt1 = new Date(dt.setDate(dt.getDate() - 7));

         let labels = this.Get_Label();

         let dt_arr = this.Get_Data(dt1,labels);

         let sort_dt = this.Sort_data(labels,dt_arr,cl_date,me);

         let svg = d3.select("#showdata").selectAll("svg")
            .data(Object.keys(sort_dt))
            .enter().append("svg")
            .attr("width",width)
            .attr("height",height)
            .append("g");
         
         
         svg.append("text")
            .attr("transform", "translate(18," + 15 + ")")
            .text(function(d){return d;});

         let rect = svg.selectAll(".item")
            .data(function(d){return sort_dt[d];})
            .enter().append("rect")
            .attr("class", "item")
            .attr("width", cellSize)
            .attr("height", cellSize)
            .attr("x", function(d,i) { return (10+i)*cellSize; })
            .style("fill",function(d){ return linear(d);});

         rect.append("title")
            .text(function(d) { return d; });
      },
      Change_Date(date){
         let [year_1,month_1,date_1] = [new Date(date).getFullYear(),new Date(date).getMonth()+1,new Date(date).getDate()];
         let l1 = year_1+'-'+month_1+'-'+date_1;
         return l1;
      },
      Get_Label(){
         let arr = Object.keys(window.store['2007-1-2'].data);
         arr.splice(arr.indexOf('latent_dim1'), 1);
         arr.splice(arr.indexOf('latent_dim2'), 1);
         arr.splice(arr.indexOf('x'), 1);
         arr.splice(arr.indexOf('y'), 1);
         arr.splice(arr.indexOf('position'), 1);
         arr.splice(arr.indexOf('日期'), 1);
         return arr ;
      },
      Get_Data(dt1,labels){
         let obj = {};
         for( let i = 0 ; i < labels.length ; i++ ){
            let key = [];
            let dt = new Date(dt1);
            let date = this.Change_Date(dt);
            for( let j = 0 ; j <= 14 ; j++ ){
               if( date in window.store ){
                  key.push(window.store[date].data[labels[i]]);
               }
               date = this.Change_Date(dt.setDate(dt.getDate()+1));
            }
            obj[labels[i]] = key;
         }
         return obj ;
      },
      Sort_data(labels,dt_arr,cl_date,me){ 
         let obj = {};
         let Obj_Map = {};
         let arr = [];
         for( let i = 0 ; i < labels.length ; i++ )
         {
            let label = labels[i];
            let base = window.store[me.Change_Date(cl_date)].data[label];
            let sz = dt_arr[label].length;
            let sum = 0 ;
            obj[label] = [];
            for( let j = 0 ; j < sz ; j++ ){
               let dis = (base === "0.0")? (dt_arr[label][j] - base):(dt_arr[label][j] - base)/Math.abs(base);
               dis = dis.toFixed(3);
               if ( dis > 0.1)
                  dis = 0.1;
               if ( dis < -0.1 )
                  dis = -0.1; 
               sum += Math.abs(dis);
               obj[label].push(dis);
            }
            sum = sum.toFixed(3);
            arr.push([label,sum]);
         }
         arr.sort((a,b)=>{ return b[1] - a[1];});
         for(let i = 0 ; i < 20 ; i++ ){  
            if( arr[i][1] === 0 ){
               break;
            }
            Obj_Map[arr[i][0]] = obj[arr[i][0]];
         }

         return Obj_Map;
      },
   }
  }
</script>
<style>
.item {
   fill: #fff;
   stroke: #ccc;
}
text{
   font-size: 100%;
}
</style>