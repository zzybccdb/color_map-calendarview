<template>
   <div class="content">
      <div class="container-fluid">
         <div class="row">
            <div class="col-12">

               <colormap :display="display">
                  <div slot="header">
                     <h4>Colormap</h4>
                  </div> 
                  <div id="cm" slot="body">
                     <canvas id="pixiCanvas" width="512" height="512"></canvas>
                  </div>  
                  <div slot="footer">
                     <button type="submit" class="btn btn-info btn-fill float-center" @click.prevent="draw_calendar">
                           Determine
                     </button>
                  </div>     
               </colormap>

               <showdata v-if="Datadispaly" v-on:father="showdatasay" :date="clickdate">
               </showdata> 

               <Calendar v-if="calendar_display" v-on:father="calendarsay" :yb="yb" :yd="yd" :sz="sz" >
                  <div slot="header">
                     <h4>Calendarview</h4>
                  </div>
               </Calendar>

            </div>
         </div>
      </div>
   </div>
</template>
<script>
   import VuePlotly from '@statnett/vue-plotly'
   import colormap from 'src/components/UIComponents/Cards/draw_colormap.vue'
   import Calendar from 'src/components/UIComponents/Cards/calendarview.vue'
   import showdata from 'src/components/UIComponents/Cards/show_data.vue'
   export default{
      components: {
         colormap,
         Calendar,
         showdata
      },
      data(){
         return{
            calendar_display : false,
            yb: 2007,
            yd: 2017,
            sz: window.size,
            Datadispaly: false,
            clickdate : 0,
            display:false,
         }
      },
      methods:{
         draw_calendar(){
            this.yb = 2007;
            this.yd = 2017;
            this.sz = window.size;
            colormap.methods.remap();
            if (this.calendar_display)
               Calendar.methods.changefill(d3v4);
            this.calendar_display = true;
            this.display = true;
         },
         calendarsay: function (date,display){
            if(display){
               this.clickdate = date;
               if (this.Datadispaly)
                  showdata.methods.initial(d3v4,this.clickdate);
               this.Datadispaly = display;
            }
            else{
               this.Datadispaly = display;
            }
         },  
         showdatasay: function (date){
         }, 
      }
   }
</script>