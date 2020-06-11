<template>
    <div>
        
        <form @submit.prevent="editarTweet(content)" v-if="editarActivo">
            <h3>Editars Tweet</h3>
            <textarea v-model="content" cols="30" rows="10" class="form-control mb-2" placeholder="Escriba su Tweet"></textarea>
            <button type="submit" class="btn btn-success">Guardar</button>
            <button @click="cancelarEdicion" type="submit" class="btn btn-danger">Cancelar</button>
        </form>

        <form @submit.prevent="agregar" v-else>
            <h3>Agregar Tweet</h3>
            <textarea v-model="content" cols="30" rows="10" class="form-control mb-2" placeholder="Escriba su Tweet"></textarea>
            <button type="submit" class="btn btn-primary">Agregar</button>
        </form>

        <hr class="mt-3">

        <h3>Listado de Tweets</h3>
        <ul class="list-group my-3">
            <li  class="list-group-item mb-4" v-for="(item, index) in tweets" :key="index" >
                 <span class="badge badge-primary float-right">
                    {{item.created_at}}
                </span>
                <p><span style="font-weight: bold">Autor: </span> {{user.name}}</p>
                <p>{{item.content}}</p>
                <button class="btn btn-danger btn-sm" @click="eliminarTweet(item, index)">Eliminar</button>
                <button class="btn btn-warning btn-sm" @click="editarFormulario(item)">Editar</button>
            </li>
        </ul>
    </div>
</template>

<script>
let user = document.head.querySelector('meta[name="user"]')

export default {
    data(){
        return{
            tweets: [],
            content: '',
            editarActivo: false,
            id : 0
        }
    },
    created(){
        axios.get('/tweets')
        .then(res =>{
            this.tweets = res.data
        })
    },
    computed:{
        user(){
            return JSON.parse(user.content)
        }
    },
    methods: {
        agregar(){
            if(this.content.trim() === ''){
                alert('Debes completar todos los campos antes de guardar');
              return;
            }
            const params = this.content;
            
            this.content = ''; 
            //console.log(params)
            axios.post('/tweets', {content: params})
                 .then(res =>{
                    this.tweets.push(res.data)
                     
                 }).catch(error => {
                     console.log(error)
                 })

                  axios.get('/tweets')
                  .then(res =>{
                     this.tweets = res.data
                   })
        },
        editarFormulario(item){
            this.editarActivo = true;
            this.content = item.content
            this.id = item.id
        },
        editarTweet(item){
            console.log(item)
            const params = {content : item}
            
            console.log(params)
            axios.put(`/tweets/${this.id}` , params)
                 .then(res => {
                    this.editarActivo = false;
                    const index = this.tweets.findIndex(item => item.id === item.id)
                    this.tweets[index] = res.data;
                    this.content = '';
                    axios.get('/tweets')
                            .then(res =>{
                                this.tweets = res.data
                            })
                 })
        },
        cancelarEdicion(){
            this.editarActivo = false;
            this.content = '';
        },
        eliminarTweet(item, index){
            axios.delete(`/tweets/${item.id}`)
                .then(() =>{
                    this.tweets.splice(index, 1)
                })
        }
    }
}
</script>