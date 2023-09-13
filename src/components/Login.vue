<template>
  <v-card>
    <v-card-title style="font-size: large;">เข้าสู่ระบบ</v-card-title>
    <v-card-text>
      <v-form
        ref="loginForm"
        v-model="valid"
        lazy-validation
  >
        <v-text-field
          v-model="name"
          :counter="20"
          :rules="nameRules"
          label="ชื่อผู้ใช้งาน"
          required
          outlined
        ></v-text-field>

        <v-text-field
          v-model="password"
          :rules="passwordRules"
          label="รหัสผ่าน"
          required
          outlined
        ></v-text-field>

        <v-btn
          :disabled="!valid"
          color="success"
          class="mr-4"
          @click="Login()"
          block
        >
          เข้าสู่ระบบ
        </v-btn>
      </v-form>
    </v-card-text>
  </v-card>
</template>

<script>
  export default {
    data: () => ({
      valid: true,
      name: '',
      nameRules: [
        v => !!v || 'กรุณากรอกชื่อผู้ใช้งาน',
        v => (v && v.length <= 20) || 'กรุณากรอกชื่อผู้ใช้งานไม่เกิน 20 ตัวอักษร',
      ],
      password: '',
      passwordRules: [
        v => !!v || 'กรุณากรอกรหัสผ่าน',
      ],
    }),
    methods:{
      Login(){
        if (this.$refs.loginForm.validate()){
          localStorage.setItem('username',this.name)
          this.$EventBus.$emit('getUsername')
          this.$router.push('/')
        }else{
          alert('ไม่ผ่าน')
        }
      }
    }
  }
</script>

<style>

</style>