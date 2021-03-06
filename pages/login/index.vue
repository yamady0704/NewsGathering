<template>
  <div class="md-layout md-alignment-center-center" style="height: 100vh;">
    <md-card class="md-layout-item md-size-50">
      <md-card-header>
        <div class="md-title">ログイン</div>
      </md-card-header>

      <form @submit.prevent="validateForm">
        <md-card-content>
          <md-field md-clearable :class="getValidationClass('email')">
            <label for="emal">メールアドレス</label>
            <md-input :disabled="loading" type="email" name="email" id="email" autocomplete="email"
            v-model="form.email" />
            <span class="md-error" v-if="!$v.form.email.required">メールアドレスは必須です</span>
            <span class="md-error" v-else-if="!$v.form.email.email">不正なメールアドレスです</span>
          </md-field>
          <md-field :class="getValidationClass('password')">
            <label for="password">パスワード</label>
            <md-input :disabled="loading" type="password" name="password" id="password"
            autocomplete="password" v-model="form.password" />
            <span class="md-error" v-if="!$v.form.password.required">パスワードは必須です</span>
            <span class="md-error" v-else-if="!$v.form.password.minLength">パスワードは6文字以上で入力してください</span>
            <span class="md-error" v-else-if="!$v.form.password.maxLength">パスワードは20文字以内で入力してください</span>
          </md-field>
        </md-card-content>

        <md-card-actions>
          <md-button to="/register">ユーザー登録</md-button>
          <md-button class="md-primary md-raised" type="submit" :disabled="loading">ログイン</md-button>
        </md-card-actions>
      </form>

      <md-snackbar :md-active.sync="isAuthenticated">
        ユーザー名「{{form.email}}」でログインしました。
      </md-snackbar>
    </md-card>

    <md-button class="md-fixed md-fab-bottom-right md-fab md-primary" @click="$router.go(-1)">
      <md-icon>arrow_back</md-icon>
    </md-button>
  </div>
</template>

<script>
  import { validationMixin } from 'vuelidate';
  import { required, email, minLength, maxLength } from 'vuelidate/lib/validators';

  export default {
    middleware: 'auth',
    mixins: [validationMixin],
    data: () => ({
      form: {
        email: '',
        password: ''
      }
    }),
    validations: {
      form: {
        email: {
          required,
          email
        },
        password: {
          required,
          minLength: minLength(6),
          maxLength: maxLength(20)
        }
      }
    },
    computed: {
      loading() {
        return this.$store.getters.loading;
      },
      isAuthenticated() {
        return this.$store.getters.isAuthenticated;
      }
    },
    watch: {
      isAuthenticated(value) {
        if (value) {
          setTimeout(() => this.$router.push('/'), 2000);
        }
      }
    },
    methods: {
      validateForm() {
        this.$v.$touch();
        if (!this.$v.$invalid) {
          this.loginUser();
        }
      },
      async loginUser() {
        await this.$store.dispatch('authenticateUser', {
          action: 'login',
          email: this.form.email,
          password: this.form.password,
          returnSecureToken: true
        })
      },
      getValidationClass(fieldName) {
        const field = this.$v.form[fieldName];
        if (field) {
          return {
            "md-invalid": field.$invalid && field.$dirty
          }
        }
      }
    }
  }
</script>