<template>
    <v-row>
        <v-col cols="10" offset="1" >
            <v-card class="mb-2">
                <v-toolbar color="primary" dark>Edit</v-toolbar>
                <v-card-text>
                    <v-breadcrumbs :items="breadcrumbs" class="pa-0" />
                    <v-form ref="form">
                        <v-text-field
                            name="fullname"
                            label="Full Name"
                            type="text"
                            :rules="rules.fullname" 
                            v-model="form.fullname"/>
                        <v-text-field
                            name="email"
                            label="Email"
                            type="email" 
                            :rules="rules.email" 
                            v-model="form.email"
                            @keydown="checkEmailExist"/>
                        <v-text-field
                            name="password"
                            label="Password"
                            type="password"
                            :rules="rules.password" 
                            v-model="form.password"/>
                        <v-text-field
                            name="retype_password"
                            label="Re-Password"
                            type="password" 
                            :rules="rules.retype_password" 
                            v-model="form.retype_password"/>
                        <v-select
                            v-model = "form.role"
                            :items="roles"
                            label = "Role"
                        ></v-select>

                        
                    </v-form>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn 
                        @click="onSubmit" 
                        color="primary"
                        :disabled="isDisable">
                        <span v-if="!isDisable">Update</span>
                        <v-progress-circular
                        v-else
                        color="primary"
                        indeterminate>
                        </v-progress-circular>
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-col>
    </v-row>
</template>

<script>
export default({
    middleware: ['authenticated'],
    asyncData ({ params }) {
        return {
            id: params.id
        }
    },
    data() {
        return {
            breadcrumbs: [
                { text: 'users', to: '/users', disabled: false, exact: true},
                { text: 'Create', disabled: true}
            ],
            emailExist: false,
            isDisable: false,
            roles: ['employee', 'cashier','admin'],
            form: {
                fullname:'',
                email:'',
                password:'',
                retype_password:'',
                role: '',
            },
            rules: {
                role: [
                    v => !!v || this.$t('FIELD_REQUIRED', { field: 'Role'}),
                ],
                fullname: [
                    v => !!v || this.$t('FIELD_REQUIRED', { field: 'Fullname'}),
                ],
                email: [
                    v => !!v || this.$t('FIELD_REQUIRED',{ field: 'Email'}),
                    v => /.+@.+/.test(v) || this.$t('EMAIL_INVALID',{ field: 'Email'}),
                    v => !this.emailExist || this.$t('EMAIL_EXIST'),
                ],
                password: [
                    //v => !!v || this.$t('FIELD_REQUIRED', { field: 'Password'}),
                    v => v.length == 0 || v.length >= 6 || this.$t('FIELD_MIN', { field: 'Password', min: 6 }),
                ],
                retype_password: [
                    v => v === this.form.password || this.$t('FIELD_CONFIRM', { field: 'Password', confirm: 'Re-Password' }),
                ],
            }
            
        }
    },
    methods: {
        checkEmailExist (){
            this.emailExist = false;
        },
        fetchData (){
            this.$axios.$get(`http://localhost:3000/users/${this.id}`)
            .then(response => {
                this.form.fullname = response.user.fullname;
                this.form.email = response.user.email;
                this.form.role = response.user.role;
            })
            .catch(error => {
                //Redirect to users
                this.$router.push({
                        name: 'users___' + this.$i18n.locale,
                        params: { message: 'ID_NOT_FOUND'}
                    })
            })
        },
        onSubmit() {
            if(this.$refs.form.validate()){
                this.isDisable = true

                this.$axios.$put(`http://localhost:3000/users/${this.id}`, this.form)
                .then(response => {
                    this.isDisable = false

                    //REDIRECT TO USERS
                    this.$router.push({
                        name: 'users___' + this.$i18n.locale,
                        params: { message: 'UPDATE_SUCCESS', fullname: this.form.fullname }
                    })
                })
                .catch(error => {
                    if(error.response.data.message == "EMAIL_EXIST") {
                        this.emailExist = true
                        this.$refs.form.validate()
                    }
                    this.isDisable = false
                })
            }
        }
    },
    mounted() {
        this.fetchData()
    }

})  
</script>