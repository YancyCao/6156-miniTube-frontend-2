<template>
    <div class="form-container">
        <el-form :model="form" label-width="300px">
            <el-form-item label="I have an account!">
                <el-switch v-model="form.login" />
            </el-form-item>
            <el-form-item label="Email Address">
                <el-input v-model="form.email" />
            </el-form-item>
            <el-form-item label="Verification Code" class="verification_code">
                <el-input v-model="form.code" class="input-with-button" />
                <el-button @click="onSendCode">Send Verification Code</el-button>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="onSubmit">{{ form.login ? "Login" : "Create" }}</el-button>
            </el-form-item>
        </el-form>
    </div>
   
</template>

<script lang="js" setup>
import { ElMessage } from 'element-plus'
import { inject, reactive, ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'

const router = useRouter()
const globalState = inject('globalState')
const form = reactive({
    login: false,
    email: '',
    code: '',
})


const onSubmit = async () => {
    if (form.login) {
        const email = form.email
        const test = (await axios.post(`${globalState.origin.value}/verify-login/`, { email: email, code: form.code })).data
        if (test.status === 'login successful') {
            globalState.username.value = email
            router.push({
                name: 'PersonalPage',
                params: { UserName: globalState.username.value },
            })
        }
        ElMessage({
            message: test.status,
            type: test.status === 'login successful' ? 'success' : 'error',
        })
    } else {
        const test = (await axios.post(`${globalState.origin.value}/verify/`, { email: form.email, code: form.code })).data
        ElMessage({
            message: test.status,
            type: test.status === 'verified' ? 'success' : 'error',
        })
    }
}

const onSendCode = async () => {
    if (form.login) {
        const test = (await axios.post(`${globalState.origin.value}/login/`, { email: form.email })).data
        ElMessage({
            message: test.status,
            type: test.status === 'verification email sent' ? 'success' : 'error',
        })
    } else {
        const test = (await axios.post(`${globalState.origin.value}/register/`, { email: form.email })).data
        ElMessage({
            message: test.status,
            type: test.status === 'subscription email sent' || test.status === 'verification code sent' ? 'success' : 'error',
        })
    }
}

</script>

<style scoped>
.form-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    background-color: #fff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.verification_code {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
}

.input-with-button {
    flex: 1;
    padding-right: 10px;
}

el-button {
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s ease;
}

el-button:hover {
    background-color: #2980b9;
}


@media only screen and (max-width: 600px) {
    .form-container {
        padding: 10px;
    }
}

</style>
