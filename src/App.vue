<template>
<div class="container">
  <Header @show-taskForm="showTask" title="Task Tracker" :showAddTask="showTaskForm"/>
  <div v-show="showTaskForm">
    <AddTask @add-task="addTask" />
  </div>
  <Tasks @toggle-remainder="toggleRemainder" @delete-task="onTaskDelete" :tasks="tasks"/>
</div>
  
</template>

<script>

import Header from './components/Header';
import Tasks from './components/Tasks';
import AddTask from './components/AddTask'
export default {
  name: "App",
  components: {
    Header,
    Tasks,
    AddTask,
  },
  data(){
    return{
      tasks:[],
      showTaskForm: false,
    }
  },
  methods:{
    showTask(){
      this.showTaskForm=!this.showTaskForm
    },
    async addTask(task){
      const res = await fetch('http://localhost:5000/tasks',{
        method:'POST',
        headers:{
          'Content-type' : 'application/json',
          
        },
        body: JSON.stringify(task)
      })
      const data = await res.json()
      this.tasks = [...this.tasks,data]
    },
    async onTaskDelete(id){

        if(confirm('Are you sure to remove?')){
          const res = await fetch(`http://localhost:5000/tasks/${id}`,{
            method:'DELETE'
          })
          res.status === 200 ? (this.tasks = this.tasks.filter((task)=>task.id !==id)): alert('Error deleting task')
            
        }
        
    },
    async toggleRemainder(id){
      const taskToggle = await this.fetchTask(id)
      const upgrade = {...taskToggle, remainder:!taskToggle.remainder}

       const res = await fetch(`http://localhost:5000/tasks/${id}`,{
         method: 'PUT',
         headers:{
           'Content-type':'application/json'
         },
         body:JSON.stringify(upgrade)
       })
      const data = await res.json()
      this.tasks = this.tasks.map((task)=>
      task.id === id ? {...task, remainder : 
      data.remainder} : task)
    },
    async fetchData(){
      const res = await fetch('http://localhost:5000/tasks')
      const data = await res.json()
      return data
    },
    async fetchTask(id){
      const res = await fetch(`http://localhost:5000/tasks/${id}`)
      const data = await res.json()
      return data
    }
  },
  async created(){
    this.tasks = await this.fetchData()
  }
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400&display=swap');
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
body {
  font-family: 'Poppins', sans-serif;
}
.container {
  max-width: 500px;
  margin: 30px auto;
  overflow: auto;
  min-height: 300px;
  border: 1px solid steelblue;
  padding: 30px;
  border-radius: 5px;
}
.btn {
  display: inline-block;
  background: #000;
  color: #fff;
  border: none;
  padding: 10px 20px;
  margin: 5px;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
  font-size: 15px;
  font-family: inherit;
}
.btn:focus {
  outline: none;
}
.btn:active {
  transform: scale(0.98);
}
.btn-block {
  display: block;
  width: 100%;
}
</style>
