<template>
   <div class="content">
      <div class="container-fluid">
         <div class="row">
            <div class="col-12">
               <Calendar>
                  <div slot="header">
                     <h4>calendarview</h4>
                  </div>
               </Calendar>
            </div>
         </div>
      </div>
   </div>
</template>
<script>
   import Calendar from 'src/components/UIComponents/Cards/calendarview.vue'
   export default{
      components: {
         Calendar,
      }
   }
</script>