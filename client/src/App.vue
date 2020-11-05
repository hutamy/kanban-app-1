<template>
  <div>
    <!-- login -->
    <LoginPage 
      v-if="pageName == 'LoginPage'"
      @changePage="changePage"
      @loginUser="loginUser">
    </LoginPage>

    <!-- register -->
    <RegisterPage 
      v-else-if="pageName == 'RegisterPage'"
      @createAccount="registerUser">
    </RegisterPage>

    <!-- homepage -->
    <HomePage 
      v-else-if="pageName == 'HomePage'"
      :tasks="tasks"
      :category="category"
      @changePage="changePage"
      @toEdit="toEdit"
      @toDelete="toDelete">
    </HomePage>

    <!-- addpage -->
    <AddPage 
      v-else-if="pageName == 'AddPage'"
      :categories="categories"
      @addTask="addTask">
    </AddPage>

    <!-- edit -->
    <EditPage 
      v-else-if="pageName == 'EditPage'"
      :detailTask="detailTask"
      @editPut="editPut">
    </EditPage>
  </div>
</template>

<script>
import LoginPage from './component/Login-Page'
import RegisterPage from './component/Register-Page'
import HomePage from './component/Home-Page'
import AddPage from './component/Add-Page'
import EditPage from './component/Edit-Page'
import axios from '../config/axios'

export default {
  name: 'App',
  data() {
    return {
      msg: 'Kanban App by Hutamy Triesthi',
      pageName: 'LoginPage',
      tasks : [],
      category : [],
      detailTask : null,
      id: 0,
      categories: []
    }
  },
  components: {
    LoginPage,
    RegisterPage,
    HomePage,
    AddPage,
    EditPage
  }, methods: {
    changePage (name){
      this.pageName = name
      this.allCategories()
    },
    fetchTasks (){
      const access_token = localStorage.getItem('access_token')
      axios({
        url: '/tasks',
        method: 'GET',
        headers: {
          access_token: access_token
        }
      })
      .then(tasks => {
        tasks.data.forEach(el => {
          this.category.push(el.Category)
        });
        this.tasks = tasks.data
      })
      .catch(err => {
        console.log(err.response)
      })
    },
    registerUser (payload) {
      axios({
        url: '/register',
        method: 'POST',
        data : {
          name: payload.name, 
          email: payload.email,
          password: payload.password
        }
      })
      .then(user => {
        const access_token = localStorage.setItem('access_token', user.data.access_token)
        this.fetchTasks()
        this.changePage('HomePage')
      })
      .catch(err => {
        console.log(err.response)
      })
    },
    loginUser(payload) {
      axios({
        url: '/login',
        method: 'POST',
        data: {
          email: payload.email,
          password: payload.password
        }
      })
      .then(user => {
        const access_token = localStorage.setItem('access_token', user.data.access_token)
        this.fetchTasks()
        this.changePage('HomePage')
      })
      .catch(err => {
        console.log(err.response)
      })
    },
    allCategories() {
      const access_token = localStorage.getItem('access_token')
      axios({
        url: '/categories',
        method: 'GET',
        headers: {
          access_token
        }
      })
      .then(category => {
        this.categories = category.data
      })
      .catch(err => {
        console.log(err.response)
      })
    },
    addTask (payload) {
      const access_token = localStorage.getItem('access_token')
      this.allCategories()
      axios({
        url: '/tasks',
        method: 'POST',
        headers: {
          access_token
        },
        data: {
          title: payload.title,
          description: payload.description,
          CategoryId: payload.CategoryId
        }
      })
      .then(task => {
        this.fetchTasks()
        this.changePage('HomePage')
      })
      .catch(err => {
        console.log(err.response)
      })
    },
    toEdit (payload){
      this.detailTask = payload
      this.pageName = 'EditPage'
      this.id = payload.id
    },
    editPut(payload) {
      const access_token = localStorage.getItem('access_token')
      const id = this.id
      axios({
        url: `/tasks/${id}`,
        method: 'PUT',
        headers: {
          access_token
        },
        data: {
          title: payload.title,
          description: payload.description,
          CategoryId: payload.CategoryId
        }
      })
      .then(task => {
        this.fetchTasks()
        this.changePage('HomePage')
      })
      .catch(err => {
        console.log(err)
      })
    },
    editPatch(){
      const access_token = localStorage.getItem('access_token')
      const id = req.params.id
      axios({
        url: `/tasks/${id}`,
        method: 'PATCH',
        headers: {
          access_token
        },
        data: {
          CategoryId
        }
      })
      .then(task => {

      })
      .catch(err => {
        console.log(err)
      })
    },
    toDelete(payload){
      this.deleteTask(payload.id)
    },
    deleteTask (id) {
      const access_token = localStorage.getItem('access_token')
      axios({
        url: `/tasks/${id}`,
        method: 'DELETE',
        headers: {
          access_token
        }
      })
      .then(data => {
        console.log(data)
        this.fetchTasks()
        this.changePage('HomePage')
      })
      .catch(err => {
        console.log(err)
      })
    }
  },
  created() {
    const access_token = localStorage.getItem('access_token')
    if(!access_token){
      this.pageName = 'LoginPage'
    }else{
      this.pageName = 'HomePage'
      this.fetchTasks()
    }
  }
}
</script>

<style>

</style>