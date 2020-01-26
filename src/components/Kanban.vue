<template>
  <div class="drag-container">
    <ul class="drag-list">
      <li v-for="stage in stages" class="drag-column" :class="{['drag-column-' + stage]: true}" :key="stage">
        <span class="drag-column-header">
          <slot :name="stage">
            <h2>{{ stage }}</h2>
          </slot>
        </span>
        <div class="drag-options"></div>
        <ul class="drag-inner-list" ref="list" :data-status="stage">
          <li class="drag-item" v-for="block in getBlocks(stage)" :data-block-id="block.id" :key="block.id">
            <slot :name="block.id">
              <strong>{{ block.title }}</strong>
              <div>{{ block.desc }}</div>
              <div>{{ block.name }}</div>
              <div>{{ block.desig }}</div>
<<<<<<< HEAD
              <div><strong>{{ block.percentage }}%</strong></div>
=======
              <div>{{ block.percentage }}</div>
>>>>>>> 7e388f9d121c9b9240f2a1f121ce5ab879130b16
              <button class="btn-sm btn btn-danger fas fa-trash" @click="deleteTask(block)"></button>
              <button class="btn-sm btn btn-primary fas fa-edit" @click="editModal(block)"></button>
            </slot>
          </li>
        </ul>
        <div class="drag-column-footer">
            <slot :name="`footer-${stage}`"></slot>
        </div>
      </li>
    </ul>
    <div class="modal fade" id="editTask" tabindex="-1" role="dialog" aria-labelledby="editTask" aria-hidden="true">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="exampleModalLabel">Add Item</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
          	<label>New Percentage:</label>
          	<input type="number" min="0" max="100" v-model="percentage" placeholder="Enter New Percentage">
          </div>
          <div class="modal-footer">
          	<button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
            <button type="submit" class="btn btn-primary" @click="changePercentage()">Submit</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import dragula from 'dragula';

  export default {
    name: 'KanbanBoard',

    props: {
      stages: {},
      blocks: {},
    },
    data() {
      return {
      	id: '',
      	percentage: ''
      };
    },

    computed: {
      localBlocks() {
        return this.blocks;
      },
    },

    methods: {
      getBlocks(status) {
        return this.localBlocks.filter(block => block.status === status);
      },
      
      
      deleteTask(block){
      	var self=this;
      	//var status=0;
      	if(confirm("Confirm Deletion of Task")){
      		axios.post('api/deleteKanbanProject', {id:block.id, admin_id:block.admin_id})
      		.then(function(response){
      			let status=response.data;
      			//console.log(status);
      			if(status==200){
      				toast.fire({
              	type:'success',
              	title:'Task Removed'
            	});
            	self.change();
      			}
      			else if(status==500){
      				toast.fire({
              	type:'error',
              	title:'Operation Not Permitted'
            	})
      			}
      			else{
      				toast.fire({
              	type:'error',
              	title:'Task Not Found'
            	})
      			}
      		});
      	}
      },
      
      editModal(block){
      	$('#editTask').modal('show');
      	var self=this;
      	self.id=block.id;
      	self.percentage=block.percentage;
      },
      changePercentage(){
      	var self=this;
      	axios.post("api/updateKanbanProject",  {id: self.id, percentage: self.percentage})
      		.then(function(response){
      			let status=response.data;
      			//console.log(status);
      			if(status==200){
      				toast.fire({
              	type:'success',
              	title:'Percentage Changed'
            	});
            	self.change();
      			}
      			else if(status==500){
      				toast.fire({
              	type:'error',
              	title:'Operation Not Permitted'
            	})
      			}
      			else{
      				toast.fire({
              	type:'error',
              	title:'Task Not Found'
            	})
      			}
      		});
      },
      change(){
      	var self=this;
        self.$emit('change','change');
      }
      
    },

  updated() {
    this.drake.containers = this.$refs.list;
  },
  mounted() {
    this.drake = dragula(this.$refs.list)
      .on('drag', (el) => {
        el.classList.add('is-moving');
      })
      .on('drop', (block, list) => {
        let index = 0;
        for (index = 0; index < list.children.length; index += 1) {
          if (list.children[index].classList.contains('is-moving')) break;
        }
        this.$emit('update-block', block.dataset.blockId, list.dataset.status, index);
      })
      .on('dragend', (el) => {
        el.classList.remove('is-moving');

        window.setTimeout(() => {
          el.classList.add('is-moved');
          window.setTimeout(() => {
            el.classList.remove('is-moved');
          }, 600);
        }, 100);
      });
  }
  };
</script>

