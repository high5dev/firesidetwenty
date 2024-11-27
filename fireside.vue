<template>
    <div class="glass-container mt-4" style="max-width: 500px;">
        <div v-if="!namesAreDone" class="p-4" style="max-width: 400px;">
            <h3 class="mb-4 fw-bold">Enter your names</h3>
            <hr class="border-2 border-secondary rounded-2" />
            <div class="row">
                <div class="col-md-12 mb-4">
                    <div class="mb-1">Name:</div>
                    <input type="text" name="name1" maxlength="20" v-model="name1" class="form-control">
                </div>
                <div class="col-md-12">
                    <div class="mb-1">Name:</div>
                    <input type="text" name="name2" maxlength="20" v-model="name2" class="form-control">
                </div>
            </div>
            <div>
                <button class="btn btn-custom mt-5 w-100" type="button" v-on:click="namesDone">Start</button>
            </div>
        </div>

        <div v-if="namesAreDone">
            <div v-if="!opened">
                <div v-if="questionNum <= highestQuestionNumber" class="box">
                    <h3 class="text-center" style="color: #d8dbd8;">Open Question {{ questionNum }}</h3>
                    <div>
                        <a href="#" v-on:click="openQuestion">
                            <img class="envelope" src="/firesidetwenty/envelope.png"
                                style="filter: drop-shadow(1px 1px 4px #000000); box-shadow: none !important;" />
                        </a>
                    </div>
                </div>
                <div v-else style="display:flex; flex-direction: column; justify-content: center; align-items: center;">
                    <div style="color: #d8dbd8; font-size:1.2rem;">
                        You have reached the end of the questions.<br />
                        Thank you and have a lovely day!
                    </div>
                    <div>
                        <a class="btn btn-danger btn-sm mt-4" href="/">Back Home</a>
                    </div>
                </div>
            </div>
            <div id="question" v-if="currentQuestion != null" class="box">
                <h3 class="mb-3" style="font-size: 1.6rem; text-align: center; width:100%; font-weight: 600;">
                    {{ currentQuestion.title }}
                    <br />
                    <span style="font-size:1.1rem;">Question {{ questionNum }}</span>
                </h3>
                <div class="mb-5 text-center">
                    {{ currentQuestion.questionText }}
                </div>
                <div class="answer-buttons"
                    style="display: flex; align-items: center; flex-direction: column; justify-content: center; gap: 1rem;">

                    <div class="answer-btn" v-on:click="leftAnswered = !leftAnswered"
                        v-bind:class="{ 'answered': leftAnswered }">
                        {{ name1 }} answered
                    </div>


                    <div class="answer-btn" v-on:click="rightAnswered = !rightAnswered"
                        v-bind:class="{ 'answered': rightAnswered }">
                        {{ name2 }} answered
                    </div>

                </div>
                <div class="mt-5 text-center" v-if="leftAnswered && rightAnswered">
                    <a v-on:click="closeCurrentQuestion"
                        style="cursor: pointer; text-decoration: underline; font-size: 1.4rem;">
                        Next Question
                    </a>
                </div>
                <div class="mt-5" v-if="!leftAnswered || !rightAnswered">
                    <p class="text-muted text-center">
                        <a href="#" v-if="allQuestions.filter(o => o.number == questionNum - 1).length > 1"
                            v-on:click="swapToAltQuestion" style="color: #3ec9fd;">
                            Don't like this question?
                            <br />
                            Click here to swap it with another one.
                        </a>
                    </p>
                </div>
            </div>
        </div>
    </div>
    <div style="position: fixed; bottom: 1rem; left: 0; right: 0; padding: 10px;">
        <div style="display:flex;">
            <div style="flex:1; text-align: left; margin-left:20px;">
                <a id="elapsedTime" href="#"
                    style="color: #aaa; text-decoration: none; font-weight: bold; cursor: pointer;">
                    <span id="shared-timer">00:00:00</span>
                </a>

            </div>
            <div style="flex:1; text-align: center;">
                <a href="/" style="color: #fff; text-decoration: none; font-weight: bold; cursor: pointer;">
                    Go Back
                </a>
            </div>
            <div style="flex:1; text-align: right; margin-right:20px;">
                <a id="change-background" href="#"
                    style="color: #aaa; text-decoration: none; font-weight: bold; cursor: pointer;">
                    Change Background
                </a>
            </div>
        </div>
    </div>
</template>

<script>
let allQuestions = @allQuestionsJsonRaw;

const FiresideApp = {
    data() {
        return {
            key: @key.ToJs(),
            name1: null,
            name2: null,
            namesAreDone: false,
            questionNum: 0,
            opened: false,
            leftAnswered: false,
            rightAnswered: false,
            questionAlts: null,

            allQuestions: allQuestions
        }
    },
    mounted() {
        // transfer data from startObj to data
        if (startObj.questionAlts == null) {
            let maxQ = Math.max(...allQuestions.map(o => o.number));
            startObj.questionAlts = new Array(maxQ).fill(0);
        }
        this.fillFromObj(startObj);
    },
    watch: {
        leftAnswered: async function (val) { // save on change
            await this.saveConversationStatus();
        },
        rightAnswered: async function (val) { // save on change
            await this.saveConversationStatus();
        },
    },
    methods: {
        fillFromObj: function (obj) {
            console.log('running fillFromObj');
            this.name1 = obj.name1;
            this.name2 = obj.name2;
            this.namesAreDone = !!obj.name1 || !!obj.name2; // crazy code to make sure it's a boolean
            this.questionNum = obj.questionNum;
            this.opened = obj.opened;
            this.leftAnswered = obj.leftAnswered;
            this.rightAnswered = obj.rightAnswered;
            this.questionAlts = obj.questionAlts;
        },
        async namesDone() {
            this.namesAreDone = true;
            // capitalize the names
            this.name1 = _.capitalize(this.name1);
            this.name2 = _.capitalize(this.name2);
            this.questionNum = 1; // start with the first question

            // save
            await this.saveConversationStatus();
        },
        openQuestion: async function () {
            this.opened = true;

            // save
            await this.saveConversationStatus();
        },
        closeCurrentQuestion: async function () {
            this.questionNum += 1;

            // reset here, not on openQuestion()
            this.opened = false;
            this.leftAnswered = false;
            this.rightAnswered = false;

            // save
            await this.saveConversationStatus();
        },
        swapToAltQuestion: async function () {
            // find all questions with this number
            let found = this.allQuestions.filter(o => o.number == this.questionNum);
            // find the current position of the current question
            let currentPos = this.questionAlts[this.questionNum - 1]; // // let currentPos = found.findIndex(o => o.questionText == this.currentQuestion.questionText);
            // if we are at the end, go back to the start
            if (currentPos == found.length - 1) {
                currentPos = 0;
            }
            else {
                currentPos += 1;
            }
            this.questionAlts[this.questionNum - 1] = currentPos;
            await this.saveConversationStatus();
        },
        async saveConversationStatus() {
            let savethis = {  // must match SaveJson in FiresideController
                key: this.key,
                name1: this.name1,
                name2: this.name2,
                namesAreDone: this.namesAreDone,
                questionNum: this.questionNum,
                opened: this.opened,
                allQuestions: this.allQuestions,
                leftAnswered: this.leftAnswered,
                rightAnswered: this.rightAnswered,
                questionAlts: this.questionAlts
            };

            console.debug(savethis);
            await fetch('/firesidedata/save', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(savethis)
            });
        }
    },
    computed: {
        highestQuestionNumber: function () {
            return Math.max(...this.allQuestions.map(o => o.number));
        },
        currentQuestion: function () {
            if (this.opened) {
                let alt = this.questionAlts[this.questionNum - 1]; // 0-indexed array, question 1 is position 0
                let found = this.allQuestions.filter(o => o.number == this.questionNum);
                return found.length > 0 ? found[alt] : undefined; // start with the first of the alts
            }
            else {
                return null;
            }

        }
    },
    components: {
        // 'count': loadVueComponent('/firesidetwenty/fireside.vue')
    }
};
export default FiresideApp;
</script>