<template>
<div style="background-color: white; height: 100%; width: 100%;">
  <v-container column style="background-color: white; height: 100%; width: 100%;">
    <v-layout row wrap style="width: 100%; flex: none">
      <v-flex xs3 sm1 md2 lg2>
        <img style="max-width: 100%; height: auto;" src="@/assets/logo.svg" />
      </v-flex>
      <v-flex sm5 md5 lg5></v-flex>
      <v-flex sm5 md5 lg5>
        <el-steps :active="currentStep" finish-status="success">
          <el-step title="Passo 1"></el-step>
          <el-step title="Passo 2"></el-step>
          <el-step title="Passo 3"></el-step>
        </el-steps>
      </v-flex>
    </v-layout>
    <v-layout v-loading="loading" align-center row wrap style="width: 100%; flex: 1 1 auto;">
        <!-- Art -->
        <v-flex sm12 md7 lg7>
          <transition name="slide-x-transition" mode="out-in">
            <img v-if="role == 'teacher'" key="teacher" style="max-width: 100%; height: auto;" src="@/assets/ilustra-construction.png" />
            <img v-else key="student" style="max-width: 100%; height: auto;" src="@/assets/signup.svg" />
          </transition>
        </v-flex>

        <!-- Steps -->
        <v-flex sm12 md5 lg5>
            <!-- Step 1 -->
            <v-layout column fill-width v-if="currentStep == 0">
              <div class="mb-4">
                <v-layout wrap>
                <span class="step-title">O que você faz na UFABC?</span>
                </v-layout>         
                <v-layout wrap>
                <span class="step-subtitle">Selecione uma das opções:</span>
                </v-layout>
              </div>
              <v-layout fill-height>
                <!-- Student -->
                <v-layout class="elevate-3d pa-4 border cursor-pointer mr-3" style="max-width: 160px;" @click="selectRole('student')">
                  <v-icon class="mr-2">mdi-school</v-icon>
                  <div>Aluno</div>
                </v-layout>
                <!-- Teacher -->
                <v-layout align-center class="elevate-3d pa-4 border cursor-pointer" style="max-width: 160px;" @click="selectRole('teacher')">
                  <v-icon class="mr-2">mdi-account</v-icon>
                  <div>Professor</div>
                </v-layout>
              </v-layout>
            </v-layout>

            <!-- Step 2 -->
            <v-layout column fill-width pr-4 v-if="currentStep == 1">
              <div v-if="role == 'student'">
                <div class="step-title mb-4">
                  Falta pouco para completar o seu cadastro
                </div>
                <div class="step-subtitle mb-2">Insira seu email institucional</div>
                <v-text-field
                  placeholder="joão"
                  suffix="@aluno.ufabc.edu.br"
                  v-model="studentData.email"
                  name="studentEmail"
                  v-validate="{required: true}"
                  data-vv-as="email institucional"
                  :error-messages="errors.collect('studentEmail')"
                  solo
                ></v-text-field>
                <div class="step-subtitle mb-2">Insira seu RA</div>
                <v-text-field
                  v-model="studentData.ra"
                  name="ra"
                  v-validate="{required: true}"
                  data-vv-as="RA"
                  :error-messages="errors.collect('ra')"
                  ref="ra"
                  placeholder="Ex: 11012014"
                  solo
                ></v-text-field>
                <div class="step-subtitle mb-2">Confirme seu RA</div>
                <v-text-field
                  name="ra_confirmation"
                  v-validate="'required|confirmed:ra'"
                  data-vv-as="confirmação do RA"
                  :error-messages="errors.collect('ra_confirmation')"
                  placeholder="Ex: 11012014"
                  v-model="raConfirmation"
                  solo
                ></v-text-field>
              </div>
              <div v-if="role == 'teacher'">
                <div class="step-title mb-4">
                  Estamos trabalhando nisso!
                </div>
                <div class="step-subtitle mb-2">Professor estamos construindo algumas ferramentas especiais para você, por enquanto você pode verificar sua distribuição de notas por disciplinas <span class="ufabcnext-link--text cursor-pointer">aqui</span></div>
              </div>
            </v-layout>

            <!-- Step 3 -->
            <div v-if="currentStep == 2">
              <div v-if="role == 'student'">
                <div class="step-title mb-4">
                  Enviamos um email de confirmação para 
                  <span class="ufabcnext-nav-blue--text">{{`${studentData.email}${emailSuffix}`}}</span>
                </div>
                <div>
                  <span class="cursor-pointer ufabcnext-link--text">Clique aqui</span> para acessar seu email institucional
                </div>
              </div>
            </div>
        </v-flex>
      </v-layout>

      <v-layout align-end row style="width: 100%; flex: none; height:57px;" >
        <v-flex sm7 md7 lg7>
        </v-flex>
        <v-flex v-show="currentStep != 0" sm5 md5 lg5 class="border-top" style="display: flex;">
          <v-btn flat round large class="mr-2" @click="back()">
            < Anterior
          </v-btn>
          <v-flex></v-flex>
          <v-btn style="background-color: #00EB5E; color: white;" round large @click="next()">
            Próximo >
          </v-btn>
        </v-flex>
      </v-layout>
  </v-container>
</div>
</template>

<script>
import ErrorMessage from '@/helpers/ErrorMessage'
import User from '@/services/User'

export default {
  name: 'SignupForm',
  inject: ["$validator"],
  data() {
    return {
      loading: false,
      currentStep: 0,
      role: '',
      studentData: {
        email: '',
        ra: ''
      },
      raConfirmation: '',
      emailSuffix: '@aluno.ufabc.edu.br'
    }
  },

  methods: {
      async next() {
        if (this.currentStep == 1) {
          if(await this.confirmAccount()) {
            this.currentStep++
            return
          }else {
           return 
          }
        }

        if (this.currentStep++ > 2) {
          this.currentStep = 0;
        }
      },

      back() {
        if (this.currentStep == 1) {
          this.role = ''
        }

        if (this.currentStep-- < 0) this.currentStep = 0;
      },

      selectRole(role) {
        this.role = role
        this.next()
      },

      async confirmAccount() {
        let email = this.studentData.email.concat(this.emailSuffix)
        return this.$validator.validateAll().then(async isValid => {
          if (!isValid) {
            // this.$message({
            //   type: 'error',
            //   message: ErrorMessage(err),
            // })
            console.log('Not valid')

            return false
          }

          //Call confirmation route
          try {
            this.loading = true
            let res = await User.completeSignup()
            this.loading = false
          } catch(err) {
            this.loading = false
            this.$message({
              type: 'error',
              message: ErrorMessage(err),
            })
          }

          return true
        })
    },
  }

}
</script>

<style scoped>
.step-title {
  font-size: 26px;
  font-weight: bold;
}

.step-subtitle {
  font-size: 14px;
  color: rgba(0,0,0,0.5);
}
</style>