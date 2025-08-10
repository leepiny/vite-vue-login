<script setup>
import {ref, onMounted} from "vue";
import axios from "axios";

const api = 'https://todolist-api.hexschool.io';
const viewStatus = ref({
    pageType    :'checking'
})

const signUpInfo = ref({
    email           :'',
    password        :'',
    checkPassword   :'',
    nickname        :'',
})

const loginInfo = ref({
    email     :'',
    password  :'',
})

const toDoInfo = ref({
    user  :'',
    token :''
})

const changePage = (type)=>{
    viewStatus.value.pageType = type
}

const signup = async()=>{
    if(Object.values(signUpInfo.value).every(val => val !== ''))
    {
        if(signUpInfo.value.password == signUpInfo.value.checkPassword)
        {
            try{
                const res = await axios.post(`${api}/users/sign_up`,signUpInfo.value);
                console.log(res);

                if(res.data.status)
                {   
                    cleanInfo(signUpInfo.value)
                }
            }
            catch (err){
                console.log(err);
            }
        }
        else
        {
            alert('密碼不一致');
        }
    }
}

const login = async()=>{
    if(Object.values(loginInfo.value).every(val => val !== ''))
    {
        if(loginInfo.value.email !== '' && loginInfo.value.password !== '')
        {
            try{
                const res = await axios.post(`${api}/users/sign_in`,loginInfo.value);
                console.log(res);

                if(res.data.status)
                {
                    document.cookie = `todo_token=${res.data.token}; path=/; max-age=${60 * 60 * 24 * 7}`;
                    await fetchMe();
                    cleanInfo(loginInfo.value)
                    viewStatus.value.pageType = 'toDo';
                }
            }
            catch (err){
                console.log(err);
            }
        }
    }
}

const logout = async ()=>{
    const token = document.cookie.replace(/(?:^|.*;\s*)todo_token\s*=\s*([^;]*).*$/i,"$1");
    
    try{
        const res = await axios.post(`${api}/users/sign_out`,{}, {
            headers: {
                Authorization: token
            }
        })
        console.log(res);
        
        if(res.data.status)
        {
            document.cookie = "todo_token=; path=/; max-age=0";
            cleanInfo(toDoInfo.value)
            viewStatus.value.pageType = 'login';
        }
    }
    catch (err){
        console.log(err);
    }
}

const fetchMe = async ()=>{
    const token = document.cookie.replace(/(?:^|.*;\s*)todo_token\s*=\s*([^;]*).*$/i,"$1");
    if (!token) throw new Error('no token');
    if(token != '')
    {
        const { data } = await axios.get(`${api}/users/checkout`, {
            headers: { Authorization: token }
        });
        toDoInfo.value.id = data.uid;
        toDoInfo.value.user = data.nickname;
    }
}

const cleanInfo = (name)=>{
    Object.keys(name).forEach(key => {
        name[key] = '';
    });
}

onMounted(async ()=>{
    try{
        await fetchMe();  
        viewStatus.value.pageType = 'toDo';
    }
    catch (err){
        viewStatus.value.pageType = 'login';
        console.log(err);
    }
})
</script>

<template>
    <div style="border:1px solid red;">
        <div v-if="viewStatus.pageType === 'checking'">載入中…</div>
        <div v-if="viewStatus.pageType === 'signUp'">
             <div>
                <input type="text" placeholder="email" v-model="signUpInfo.email">
            </div>
            <div>
                <input type="text" placeholder="nick name" v-model="signUpInfo.nickname">
            </div>
            <div>
                <input type="password" placeholder="password" v-model="signUpInfo.password">
            </div>
            <div>
                <input type="password" placeholder="check password" v-model="signUpInfo.checkPassword">
            </div>
            <div>
                <button type="button" @click="signup">Sign up</button>
                <br>
                <button type="button" @click="changePage('login')">Login</button>
            </div>
        </div>

        <div v-if="viewStatus.pageType === 'login'">
            <div>
                <input type="text" placeholder="email" v-model="loginInfo.email">
            </div>
            <div>
                <input type="password" placeholder="password" v-model="loginInfo.password">
            </div>
            <div>
                <button type="button" @click="changePage('signUp')">Sign up</button>
                <br>
                <button type="button" @click="login">Login</button>
            </div>
        </div>

        <div v-if="viewStatus.pageType == 'toDo'">
            <input type="text" placeholder="待辦事項">
            <div>{{ toDoInfo.user }}</div>
            <div>{{ toDoInfo.id }}</div>
            <button type="button" @click="logout">logout</button>
        </div>
    </div>
</template>

<style>

</style>