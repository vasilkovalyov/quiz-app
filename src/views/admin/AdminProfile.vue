<template>
    <div class="auth-profile-page">
        <AdminBanner 
            captionBanner="Admin"
            :userName="getName"
            typePage="Admin settings"
            image="admin-profile-image.jpg"
        />
        <section class="section-profile">
            <div class="container">
                <div class="section-profile__content">
                    <div class="section-profile__row">
                        <div class="section-profile__col section-profile__col-4">
                            <div class="profile-panel profile-photo">
                                <span class="profile-panel__caption profile-photo__caption">Profile Photo</span>
                                <ImageUpload 
                                    name="image-user"
                                    :image="userForm.image"
                                    @imageUploadHandler="imageUploadHandler"
                                />
                            </div>
                        </div>
                        <div class="section-profile__col section-profile__col-8">
                            <div class="profile-panel">
                                <div class="profile-panel__header">
                                    <span class="profile-panel__caption">Your Profile</span>
                                    <div class="btn-wrap">
                                        <FormButton :buttonClass="['v-btn__primary']" @clickHandler="handleClickSave" :loader="loaderMainForm">Save</FormButton>
                                    </div>
                                </div>
                                <div class="profile-panel__category">
                                    <span class="profile-panel__category-title">BASICS</span>
                                </div>
                                <div class="profile-panel__form" v-if="userForm">
                                    <div class="profile-panel__row">
                                        <div class="profile-panel__col profile-panel__col-6">
                                            <InputField
                                                v-model="userForm.firstName" name="first-name" placeholder="Name">
                                                <template v-slot:input-icon>
                                                    <i class="fas fa-user"></i>
                                                </template>
                                                <template v-slot:label>First Name</template>
                                            </InputField>
                                        </div>
                                        <div class="profile-panel__col profile-panel__col-6">
                                            <InputField
                                                v-model="userForm.lastName" name="last-name" placeholder="Name">
                                                <template v-slot:input-icon>
                                                    <i class="fas fa-user"></i>
                                                </template>
                                                <template v-slot:label>Last Name</template>
                                            </InputField>
                                        </div>
                                        <div class="profile-panel__col profile-panel__col-6">
                                            <InputField v-model="userForm.jobTitle" name="job" placeholder="Job Title">
                                                <template v-slot:input-icon>
                                                    <i class="fas fa-user-md"></i>
                                                </template>
                                                <template v-slot:label>Job Title</template>
                                            </InputField>
                                        </div>
                                        <div class="profile-panel__col">
                                            <InputField v-model="userForm.email" name="email" placeholder="Email" :disabled="true">
                                                <template v-slot:input-icon>
                                                    <i class="fas fa-envelope"></i>
                                                </template>
                                                <template v-slot:label>Email</template>
                                            </InputField>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="section-profile__col">
                            <div class="profile-panel disabled">
                                <div class="profile-panel__category">
                                    <span class="profile-panel__category-title">Security</span>
                                </div>
                                <div class="profile-panel__password-form">
                                    <div class="profile-panel__row">
                                        <div class="profile-panel__col profile-panel__col-4">
                                            <span class="profile-panel__caption">Change Password</span>
                                        </div>
                                        <div class="profile-panel__col profile-panel__col-8">
                                            <form class="confirm-password-form">
                                                <div class="confirm-password-form__input-wrapper">
                                                    <InputField 
                                                        v-model="securityDate.password" 
                                                        name="password" 
                                                        type="password"
                                                        placeholder="Password" 
                                                        :hasIcon="true"
                                                        sizeInput="md"
                                                    >
                                                        <template v-slot:input-icon>
                                                            <i class="fas fa-lock"></i>
                                                        </template>
                                                    </InputField>
                                                </div>
                                                <div class="confirm-password-form__input-wrapper">
                                                    <InputField 
                                                        v-model="securityDate.confirmPassword" 
                                                        name="confirm-password" 
                                                        type="password" 
                                                        placeholder="Confirm Password"
                                                        :hasIcon="true"
                                                        sizeInput="md"
                                                    >
                                                        <template v-slot:input-icon>
                                                            <i class="fas fa-lock"></i>
                                                        </template>
                                                    </InputField>
                                                </div>
                                                <div class="btn-wrap">
                                                    <FormButton :buttonClass="['v-btn__primary']" @clickHandler="handleClickSavePassword" :loader="loaderPassword">Save</FormButton>
                                                </div>
                                            </form>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
        <template v-if="isShowNotification">
            <Notification
                :message="notification.message"
                :type="notification.type"
                :timeout="2000"
                @closeNotification="closeNotification"
            />
        </template>
    </div>
</template>

<script>

import { mapGetters, mapActions } from 'vuex';

import InputField from '@/components/FormsComponents/InputField';
import FormButton from '@/components/FormsComponents/FormButton';
import ImageUpload from '@/components/ImageUpload';
import Notification from '@/components/Notification';
import AdminBanner from '@/components/AdminBanner';

export default {
    name: 'AdminProfile',

    data() {
        return {
            name: 'Vasiliy',
            userForm: {},
            securityDate: {
                password: '',
                confirmPassword: ''
            },
            loaderMainForm: false,
            loaderPassword: false,
            notification: {
                message: '',
                type: ''
            },
            isShowNotification: false,
            loadUserDate: false
        }
    },

    computed: {
        ...mapGetters(['getAuthUserDate']),

        getName() {
            if(this.loadUserDate) {
                return this.getAuthUserDate.firstName + ' ' + this.getAuthUserDate.lastName;
            }
        }
    },

    async mounted() {
        await this.$store.dispatch('onAuthUser');
        this.userForm =  { ...this.getAuthUserDate };
        this.loadUserDate = true;
        
	},

    methods: {
        ...mapActions(['updateAuthUser', 'onAuthUser']),

        async handleClickSave() {
            try {
                this.loaderMainForm = true;
                await this.updateAuthUser(this.userForm);
                this.loaderMainForm = false;
                this.initNotification(true);
            } catch(error) {
                console.log(error);
                this.loaderMainForm = false;
                this.initNotification(false);
            }
        },

        imageUploadHandler(file) {
            this.userForm.image = file;
        },

        closeNotification() {
            this.isShowNotification = false;
        },

        initNotification(flag) {
            this.isShowNotification = true;

            if(flag) {
                this.notification.message = 'Admin data save - Successfull',
                this.notification.type = 'successfull'
            } else {
                this.notification.message = 'Admin data save - Error',
                this.notification.type = 'error'
            }
        },

        handleClickSavePassword() {
            // function for change password
        }
    },

    components: {
        InputField,
        FormButton,
        ImageUpload,
        Notification,
        AdminBanner
    }
}

</script>