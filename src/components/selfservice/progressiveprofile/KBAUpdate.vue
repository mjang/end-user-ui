<template>
    <div>
        <b-form-group label-text-align="left" class="mb-0"
            v-for="(answer, key) in answers" :key="key">

            <b-form-select class="mb-3"
                v-model="selected[key].selected"
                :options="options" ></b-form-select>

            <fr-floating-label-input class="mb-3" type="text"
                v-if="selected[key].selected === customIndex"
                v-model.trim="answer.customQuestion"
                :fieldName="$t('common.user.kba.question').toLowerCase()"
                :label="$t('common.user.kba.question')"
                :validateRules="'required'"></fr-floating-label-input>

            <fr-floating-label-input class="mb-3" type="text"
                v-model.trim="answer.answer"
                :fieldName="$t('common.user.kba.answer').toLowerCase()"
                :label="$t('common.user.kba.answer')"
                :validateRules="'required'"></fr-floating-label-input>

            <hr v-if="key !== answers.length - 1">
        </b-form-group>
    </div>
</template>

<script>
    import _ from 'lodash';
    import FloatingLabelInput from '@/components/utils/FloatingLabelInput';

    /**
     * @description Selfservice stage for progressive profile, handles the case where a resourse is requested to change their KBA questions
     *
     **/
    export default {
        name: 'KBA-Update-Stage',
        props: {
            selfServiceDetails: { required: true }
        },
        $_veeValidate: {
            validator: 'new'
        },
        components: {
            'fr-floating-label-input': FloatingLabelInput
        },
        inject: ['$validator'],
        data () {
            let locale = this.$i18n.locale,
                numberOfQuestions = _.toString(this.selfServiceDetails.requirements.properties.kba.minItems),
                answers = _.times(numberOfQuestions, (n) => { return { answer: null, questionId: null, customQuestion: null }; }),
                selected = _.times(numberOfQuestions, () => { return {selected: null}; }),
                options = _.map(this.selfServiceDetails.requirements.properties.kba.questions, (question) => {
                    return { value: question.id, text: question.question[locale], disabled: false };
                }),
                customIndex = options.length + 1;

            // 'placeholder' should be first item in options array and 'custom' should be last
            options.unshift({ value: null, text: this.$t('common.user.kba.selectQuestion'), disabled: true });
            options.push({ value: customIndex, text: this.$t('common.user.kba.custom'), disabled: false });

            return {
                selected: selected,
                options: options,
                answers: answers,
                customIndex: customIndex
            };
        },
        watch: {
            selected: {
                handler (value) {
                    // create array of selected options that aren't custom
                    let toDisable = _.map(this.selected, (s) => {
                        if (s.selected !== null && s.selected !== this.customIndex) {
                            return s.selected;
                        }
                    });

                    // set any [toDisable] option to disabled
                    _.each(this.options, (o) => {
                        if (_.includes(toDisable, o.value) || o.value === null) {
                            o.disabled = true;
                        } else {
                            o.disabled = false;
                        }
                    });
                },
                deep: true
            }
        },
        methods: {
            getData () {
                let saveArray = _.map(this.selected, (s, index) => {
                    let couplet = {};

                    // if custom question
                    if (s.selected === this.customIndex) {
                        couplet.customQuestion = this.answers[index].customQuestion;
                    } else {
                        couplet.questionId = s.selected;
                    }
                    couplet.answer = this.answers[index].answer;
                    return couplet;
                });

                return { kba: saveArray };
            },
            isValid () {
                /* istanbul ignore next */
                return this.$validator.validateAll();
            }
        }
    };
</script>
