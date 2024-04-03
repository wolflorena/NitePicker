<script setup>
import { reactive, ref, watch } from "vue";
import CustomButton from "@/components/CustomButton.vue";
import DATA from "@/utils/movies_and_tvshows.json";

const results = ref(DATA);

const rules = {
  q0: (input, facts) =>
    !input ? true : input === "any" ? true : facts.q0 === input,
  q1: (input, facts) =>
    !input
      ? true
      : input === "any"
      ? true
      : checkResponseArray(input, facts.q1),
  q2: (input, facts) =>
    !input
      ? true
      : input === "any"
      ? true
      : checkResponseInterval(input, facts.q2),
  q3: (input, facts) =>
    !input
      ? true
      : input === "any"
      ? true
      : checkResponseArray(input, facts.q3),
  q4: (input, facts) =>
    !input || !facts.qm0
      ? true
      : input === "any"
      ? true
      : checkResponseInterval(input, facts.qm0),
  q5: (input, facts) =>
    !input || !facts.qt0
      ? true
      : input === "any"
      ? true
      : checkResponseInterval(input, facts.qt0),
};
const ruleKeys = Object.keys(rules);

const checkResponseArray = (response, facts) => {
  if (!Array.isArray(response)) {
    response = [response];
  }
  return response.some((element) => facts.includes(element));
};

function checkResponseInterval(response, facts) {
  const [startInterval, endInterval] = response.split("-");

  if (facts >= parseInt(startInterval) && facts <= parseInt(endInterval)) {
    return true;
  }
  return false;
}

function checkRules(entries, input, rule) {
  const res = entries.filter((entry) => {
    return rule(input, entry.facts);
  });

  if (res.length === 0) {
    return entries;
  }
  return res;
}

const state = reactive({
  questions: {
    q0: {
      text: "Do you want to watch a movie or a TV show?",
      options: ["movie", "tv-show", "any"],
    },
    q1: {
      text: "What genre are you interested in?",
      options: [
        "crime",
        "drama",
        "adventure",
        "biography",
        "action",
        "romance",
        "history",
        "fantasy",
        "sci-fi",
        "thriller",
        "horror",
        "music",
        "comedy",
        "mystery",
        "animation",
        "family",
        "any",
      ],
    },
    q2: {
      text: "Do you have any preferred production years?",
      options: [
        "1970-1980",
        "1981-1990",
        "1991-2000",
        "2001-2010",
        "2011-2020",
        "2021-2024",
        "any",
      ],
    },
    q3: {
      text: "What is your preferred age range for content?",
      options: ["18+", "16+", "13+", "any"],
    },
    q4: {
      text: "How long would you like the movie to be?",
      options: ["110-130", "131-150", "151-170", "171-190", "191-210", "any"],
    },
    q5: {
      text: "How many seasons would you like the TV series to have?",
      options: ["1-2", "3-4", "5-6", "7-8", "9-10", "any"],
    },
  },
  currentQuestionIndex: 0,
  selectedAnswers: {
    0: "",
    1: [],
    2: "",
    3: [],
    4: "",
    5: "",
  },
  showResult: false,
});

const questionsArray = ref(Object.values(state.questions));

function nextQuestion() {
  results.value = checkRules(
    results.value,
    state.selectedAnswers[state.currentQuestionIndex],
    rules[ruleKeys[state.currentQuestionIndex]]
  );
  if (state.currentQuestionIndex === 3) {
    let nextIndex;
    if (
      state.selectedAnswers[0] === "any" ||
      state.selectedAnswers[0] === "" ||
      state.selectedAnswers[0] === null
    ) {
      nextIndex = questionsArray.value.findIndex(
        (question) => question.text === state.questions.q4.text
      );
    } else if (state.selectedAnswers[0] === "tv-show") {
      nextIndex = questionsArray.value.findIndex(
        (question) => question.text === state.questions.q5.text
      );
    } else {
      nextIndex = questionsArray.value.findIndex(
        (question) => question.text === state.questions.q4.text
      );
    }

    if (nextIndex !== undefined) {
      state.currentQuestionIndex = nextIndex;
      state.showResult = false;
      return;
    }
  }

  if (
    (state.currentQuestionIndex >= questionsArray.value.length - 2 &&
      (state.selectedAnswers[0] === "movie" ||
        state.selectedAnswers[0] === "tv-show")) ||
    state.currentQuestionIndex >= questionsArray.value.length - 1
  ) {
    state.showResult = true;
  } else {
    state.currentQuestionIndex++;
  }
}

watch(
  () => state.currentQuestionIndex,
  (newVal) => {
    if (!(newVal in state.selectedAnswers)) {
      state.selectedAnswers[newVal] = null;
    }
  }
);

function resetTest() {
  state.currentQuestionIndex = 0;
  state.showResult = false;
  state.selectedAnswers = {
    0: "",
    1: [],
    2: "",
    3: [],
    4: "",
    5: "",
  };
  results.value = DATA;
}
</script>

<template>
  <div class="home">
    <img class="logo" src="../assets/logo.png" alt="" />
    <div class="questions" v-if="!state.showResult">
      <h1>
        {{ questionsArray[state.currentQuestionIndex].text }}
      </h1>
      <div class="options">
        <div
          v-for="(option, index) in questionsArray[state.currentQuestionIndex]
            .options"
          :key="index"
          class="radio-option"
        >
          <label
            :for="`option-${state.currentQuestionIndex}-${index}`"
            v-if="
              state.currentQuestionIndex === 1 ||
              state.currentQuestionIndex === 3
            "
          >
            <input
              type="checkbox"
              :value="option"
              :id="`option-${state.currentQuestionIndex}-${index}`"
              v-model="state.selectedAnswers[state.currentQuestionIndex]"
              :name="`question-${state.currentQuestionIndex}`"
              required
            />{{ option.charAt(0).toUpperCase() + option.slice(1) }}</label
          >

          <label :for="`option-${state.currentQuestionIndex}-${index}`" v-else>
            <input
              type="radio"
              :value="option"
              :id="`option-${state.currentQuestionIndex}-${index}`"
              v-model="state.selectedAnswers[state.currentQuestionIndex]"
              :name="`question-${state.currentQuestionIndex}`"
              required
            />
            {{ option.charAt(0).toUpperCase() + option.slice(1) }}</label
          >
        </div>
      </div>
      <div class="button">
        <CustomButton title="Next" @click="nextQuestion" />
      </div>
    </div>
    <div class="filtered-results" v-if="state.showResult">
      <h2>Our recommandation:</h2>
      <div class="result">
        <a
          :href="`https://www.imdb.com/find?q=${encodeURIComponent(
            results[0].details.name
          )}`"
          target="_blank"
        >
          <img
            :src="results[0].details.background"
            :alt="results[0].details.name"
          />
        </a>

        <div class="details">
          <h1>
            {{ results[0].details.name }}
            <small>{{ "(" + results[0].details.production_year + ")" }}</small>
          </h1>
          <h3>
            {{
              results[0].details.genre.replaceAll(",", " |") +
              " (" +
              results[0].details.age_range +
              ")"
            }}
          </h3>
          <p>
            {{ results[0].details.description }}
          </p>
          <p>
            {{
              results[0].details.duration ||
              results[0].details.number_seasons + " seasons"
            }}
          </p>
        </div>
        <CustomButton title="Try again" @click="resetTest" />
      </div>

      <div class="others" v-if="results.length > 1">
        <h2>You also may like...</h2>
        <div class="results">
          <div
            class="result"
            v-for="(item, index) in results.slice(1, 5)"
            :key="index"
          >
            <h3>
              {{ item.details.name }}
              <small>{{ "(" + item.details.production_year + ")" }}</small>
            </h3>
            <a
              :href="`https://www.imdb.com/find?q=${encodeURIComponent(
                item.details.name
              )}`"
              target="_blank"
            >
              <img :src="item.details.background" :alt="item.details.name" />
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.home {
  background-image: url("../assets/background.jpg");
  height: 100vh;
  color: white;

  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  .logo {
    position: absolute;
    top: 20px;
    width: 150px;
  }

  .questions {
    width: 50%;

    display: flex;
    justify-content: center;
    flex-direction: column;
    gap: 10px;

    .options {
      height: 250px;
      overflow: scroll;
      display: flex;
      flex-direction: column;
      gap: 5px;
    }
    .radio-option {
      label {
        position: relative;
        cursor: pointer;
        display: flex;
        align-items: center;
      }

      input[type="radio"],
      input[type="checkbox"] {
        appearance: none;
        background: none;
        border: 2px solid #777;
        height: 1.5em;
        width: 1.5em;
        margin-right: 10px;
        border-radius: 100%;
        vertical-align: text-bottom;
        position: relative;
      }

      input[type="checkbox"] {
        border-radius: 0;
      }

      input[type="radio"]::before,
      input[type="checkbox"]::before {
        content: "";
        position: absolute;
        margin: auto;
        left: 0;
        right: 0;
        bottom: 0;
        overflow: hidden;
        top: 0;
      }

      input[type="radio"]:checked::before {
        border: 5px solid transparent;
        border-radius: 100%;
        background: #ddb31b;
      }

      input[type="checkbox"]:checked::before {
        border-right: 5px solid #ddb31b;
        border-bottom: 6px solid #ddb31b;
        height: 90%;
        width: 30%;
        transform: rotate(50deg) translateY(-20%) translateX(-10%);
      }

      input[type="radio"]:focus,
      input[type="checkbox"]:focus {
        outline: solid 1px;
        outline-offset: 2px;
      }
    }

    .button {
      width: 100%;
      display: flex;
      justify-content: flex-end;
    }
  }

  .filtered-results {
    display: flex;
    flex-direction: column;

    .result {
      background-color: #1e1e1e;
      border-radius: 15px;
      padding: 10px;
      width: 100%;
      height: 280px;

      display: flex;
      align-items: center;
      gap: 20px;

      img {
        height: 250px;
        border-radius: 10px;
        -webkit-box-shadow: 20px 17px 54px 10px rgba(0, 0, 0, 0.53);
        -moz-box-shadow: 20px 17px 54px 10px rgba(0, 0, 0, 0.53);
        box-shadow: 20px 17px 54px 10px rgba(0, 0, 0, 0.53);
        transition: 0.4s;
        &:hover {
          transform: scale(1.1);
        }
      }

      .details {
        display: flex;
        flex-direction: column;
        justify-content: center;
        max-width: 500px;

        height: 100%;
        gap: 20px;
        position: relative;
        z-index: 2;
        text-shadow: 0 3 10 rgba(0, 0, 0, 0.7);
        h1 {
          font-family: "Rubik", Sans-serif;
          font-size: 3em;
          margin: 0;
          text-transform: uppercase;
          font-weight: 700;
          color: #ddb31b;

          small {
            font-size: 0.4em;
            color: #cccccc;
            position: relative;
            bottom: 10px;
          }
        }
        h3 {
          margin: 0;
          font-weight: 700;
        }
        p {
          font-weight: 300;
          width: 60%;
        }
      }
    }

    .others {
      display: flex;
      flex-direction: column;
    }
    .results {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 20px;

      .result {
        flex-direction: column;
        justify-content: space-around;
        align-items: center;
        gap: 0;
        margin-bottom: 0px;
        text-align: center;

        width: 20%;
        height: 250px;
        width: 200px;
        position: relative;

        img {
          height: 170px;
          border-radius: 5px;
        }

        h3 {
          color: #ddb31b;
        }
      }
    }
  }
}
</style>
