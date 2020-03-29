<template>
    <div>
        <b-jumbotron>
            <template slot="lead">
                <div v-html="currentQuestion ? currentQuestion.question : 'Loading questions...'">
                </div>
            </template>

            <hr class="my-4" />

            <div>
                <b-list-group v-if="currentQuestion">
                    <b-list-group-item 
                        v-for="(answer, index) in currentQuestion.answers" 
                        :key="index"
                        @click="selectAnswer(index)"
                        :class="getAnswerClass(index)"
                    >
                        <div v-html="answer">
                        </div>
                    </b-list-group-item>
                </b-list-group>
            </div>
            
            <div>
                <b-row class="buttons-row">
                    <b-col>
                        <b-button 
                            block
                            variant="success" 
                            href="#" 
                            :disabled="!currentQuestion" 
                            @click=moveToPreviousQuestion
                        >
                            Previous
                        </b-button>
                    </b-col>
                    <b-col>
                        <b-button 
                            block  
                            variant="success" 
                            href="#"
                            :disabled="!currentQuestion" 
                            @click=moveToNextQuestion
                        >
                            Next
                        </b-button>
                    </b-col>
                </b-row>
                <b-row>
                    <b-col>
                        <b-button 
                            id="submit-button"
                            block
                            variant="primary" 
                            href="#" 
                            :disabled="submitted || answeredQuestions < questions.length"
                            @click=submit
                        >
                            Submit
                        </b-button>
                        <b-tooltip 
                            target="submit-button" 
                            triggers="hover" 
                            
                        >
                            Please answer all the questions
                        </b-tooltip>
                    </b-col>
                </b-row>
            </div>
        </b-jumbotron>
        <b-modal id="results-modal" title="Your Results">
            <p class="my-4">You scored {{ correctAnswersCount() }} out of {{ questions.length }}</p>
        </b-modal>
    </div>

</template>

<script>
import _ from 'lodash'

export default {
    data() {
        return {
            currentQuestion: null,
            questions: [],
            questionIndex: 0,
            submitted: false,
            answeredQuestions: 0
        }
    },
    watch: {
        questionIndex: function(newQuestionIndex) {
            this.currentQuestion = this.questions[newQuestionIndex]
        }
    },
    methods: {
        correctAnswersCount() {
            return this.questions.filter(question => question.selectedAnswerIndex === question.correctAnswerIndex).length
        },
        getAnswerClass(index) {
            if (this.submitted) {
                if (index == this.currentQuestion.correctAnswerIndex) {
                    return ['correct-answer']
                } else if (index == this.currentQuestion.selectedAnswerIndex) {
                    return ['incorrect-answer']
                }
            } else {
                if (index == this.currentQuestion.selectedAnswerIndex) {
                    return ['selected-answer']
                }
            }

            return ['']
        },
        selectAnswer(index) {
            if (!this.submitted) {
                if (this.currentQuestion.selectedAnswerIndex === index) {
                    this.currentQuestion.selectedAnswerIndex = null
                    this.answeredQuestions--
                } else {
                    this.currentQuestion.selectedAnswerIndex = index
                    this.answeredQuestions++
                }
            }
        },
        submit() {
            this.submitted = true

            this.$bvModal.show('results-modal')
        },
        moveToPreviousQuestion() {
            this.questionIndex = (this.questionIndex - 1) % this.questions.length

            this.$emit('currentQuestionChanged', this.questionIndex)
        },
        moveToNextQuestion() {
            this.questionIndex = (this.questionIndex + 1) % this.questions.length

            this.$emit('currentQuestionChanged', this.questionIndex)
        }
    },
    mounted: function() {
        fetch('https://opentdb.com/api.php?amount=10', {
            method: 'get'
        })
        .then((response) => {
            return response.json()
        })
        .then((jsonData) => {
            this.questions = jsonData.results.map(rawQuestion => {
                let answers = [...rawQuestion.incorrect_answers]

                answers.push(rawQuestion.correct_answer)

                if (answers.length > 2) {
                    answers = _.shuffle(answers)
                }

                let correctAnswerIndex = answers.indexOf(rawQuestion.correct_answer)
                let question = {
                    question: rawQuestion.question,
                    answers: answers,
                    correctAnswerIndex: correctAnswerIndex,
                    selectedAnswerIndex: null
                }

                return question
            })

            this.currentQuestion = this.questions[0]
        })
    }
}
</script>

<style scoped>
.list-group {
    margin-bottom: 15px;
}

.list-group-item:hover {
    background: #EEE;
    cursor: pointer;
}

.btn {
    margin: 0 5px;
}

.selected-answer {
    background-color: lightblue !important;
}

.correct-answer {
    background-color: lightgreen !important;
}

.incorrect-answer {
    background-color: red !important;
}

/* .buttons-row div.col {
    padding-right: 0px;
} */
</style>