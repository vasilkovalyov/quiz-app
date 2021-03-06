<template>
    <section class="section-admin-default">
        <div class="container">
            <AdminSectionHeader>
                <template v-slot:title>Create Quiz Questions</template>
            </AdminSectionHeader>
            <div class="section-admin-default__body">
                 <form class="form-admin-question">
                    <FormSelect 
                        name="game-select" 
                        baseOption="Select quiz" 
                        :options="getQuestionsForSelect"
                        @onChange="onChangeSelect"
                    >
                        <template v-slot:label>Choose Quiz</template>
                    </FormSelect>
                    <InputField 
                        v-model="formData.name"
                        name="questions-name" 
                        placeholder="Input questions name" 
                    >
                        <template v-slot:label>Questions name</template>
                    </InputField>

                    <FieldOptions
                        caption="Options"
                        :countOptions="countOptions"
                        @handleChangeOptions="handleChangeOptions"
                    />
                    <FormButton :buttonClass="['v-btn__primary']" @clickHandler="formSubmit" :loader="loader">Create Question</FormButton>
                </form>
            </div>
        </div>
        <template v-if="isShowNotification">
            <Notification
                :message="notification.message"
                :type="notification.type"
                :timeout="2000"
                @closeNotification="closeNotification"
            />
        </template>
    </section>
</template>
<script>

import { clearFormObject } from '@/helpers'

import { mapGetters, mapActions } from 'vuex';

import InputField from '@/components/FormsComponents/InputField';
import FormSelect from '@/components/FormsComponents/FormSelect';
import TextAreaField from '@/components/FormsComponents/TextAreaField';
import FormButton from '@/components/FormsComponents/FormButton';
import FieldOptions from '@/components/FormsComponents/FieldOptions';
import Notification from '@/components/Notification';
import AdminSectionHeader from '@/components/AdminSectionHeader';



export default {
    name: 'CreateQuizQuestions',

    data() {
        return {
            games: [],
            formData: {
                dateCreated: '',
                name: '',
                game: {},
                questionsArray: [],
                trueAnswer: null,
            },
            countOptions: 4,
            loader: false,
            isShowNotification: false,
            notification: {
                message: '',
                type: ''
            },
        }
    },

    async mounted() {
        await this.onLoadGames();
        this.games = await this.getQuizGames;
    },

    computed: {
        ...mapGetters(['getQuizGames']),

        getQuestionsForSelect() {
            return this.getQuizGames.map((item) => ({
                'code': item.key,
                'label': item.name
            }));
        }
    },

    methods: {
        ...mapActions(['onCreateQuestionForGame', 'onLoadGames']),

        async formSubmit() {
            const keyGame = this.formData.game.code;
            const dataObject = {
                key: keyGame, 
                data: {
                    ...this.formData,
                    questionsArray: [...this.formData.questionsArray],
                    question: this.formData.name,
                    trueAnswer: this.formData.trueAnswer,
                    dateCreated: new Date().toLocaleDateString()
                }
            }

            try {
                await this.onCreateQuestionForGame(dataObject);
                this.initNotification(true);
                clearFormObject(this.formData);
            } catch(e) {
                this.initNotification(false);
                console.log(e);
            }
        },

        onChangeSelect(selectData) {
            this.formData.game = {...selectData}
        },

        handleChangeOptions(data) {
            this.formData.questionsArray = data.options;
            this.formData.trueAnswer = data.trueAnswer;
        },

        closeNotification() {
            this.isShowNotification = false;
        },

        initNotification(flag) {
            this.isShowNotification = true;

            if(flag) {
                this.notification.message = 'Quiz Question Create - Successfull',
                this.notification.type = 'successfull'
            } else {
                this.notification.message = 'Quiz Question Create - Error',
                this.notification.type = 'error'
            }
        },
    },

    components: {
        InputField,
        FormSelect,
        TextAreaField,
        FormButton,
        FieldOptions,
        Notification,
        AdminSectionHeader
    }
}
</script>

<style lang="scss">
    .form-create-quiz {
        padding: 50px;
    }
</style>