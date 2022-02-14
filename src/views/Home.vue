<template>
  <div class="home">
    <div class="title">
      <h1>なんでもクイズ</h1>
      <button @click="load_question">問題表示</button>
      <button>CSV読み込み</button>
    </div>
    <!-- 問題表示画面 -->
    <div class="question_page" :style="{display:QP_view}">
      <p class="question_type">ジャンル：一般常識</p>
      <div class="contents">
        <div class="question_area">
          <p>ー 問題 ー</p>
          <h3 class="question">{{q_sentence}}</h3>
        </div>

        <div class="answer_area">
          <div class="answer_area">
            <table>
              <tr>
                <td><p class="ans1" @click="choice_ans(answer1)">{{answer1}}</p></td>
                <td><p class="ans2" @click="choice_ans(answer2)">{{answer2}}</p></td>
              </tr>
              <tr>
                <td><p class="ans3" @click="choice_ans(answer3)">{{answer3}}</p></td>
                <td><p class="ans4" @click="choice_ans(answer4)">{{answer4}}</p></td>
              </tr>
            </table>
          </div>
        </div>
      </div>
      <input type="range" min="0" max="100" step="10" value="50"><span>{{}}</span>
    </div>

    <!-- 解説ページ -->
    <div class="commentary_page">
      <div class="result_title">
        <h2>{{result_answer}}</h2>
        <p>aa</p>
        <button @click="next_question">次の問題</button>
      </div>
    </div>
  </div>
</template>

<script>
import {ref} from 'vue'
export default {
  name: 'Home',
  setup(){
    // 問題データ
    let questions_data = ref([
      {question:"「走る」は英語で？",ans1:"run",ans2:"height",ans3:"wait",ans4:"walk", r_ans:"run"},
      {question:"日本人はどれ？",ans1:"サンチェズ",ans2:"岡村",ans3:"ジョニー",ans4:"キム", r_ans:"岡村"},
      {question:"「こんにちは」の返しは？",ans1:"さようなら",ans2:"ごめんなさい",ans3:"こんにちは",ans4:"また会いましょう", r_ans:"こんにちは"},
    ])

    let amount_corrects = ref()

    let result_answer = ref()
    
    // 問題数カウント
    let count = ref(0)

    // 画面表示管理
    let QP_view = ref("block")
    let CP_view = ref()

    // 回答ボタン
    let answer1 = ref()
    let answer2 = ref()
    let answer3 = ref()
    let answer4 = ref()
    let r_ans = ref()
    let q_sentence = ref()

    // 音源再生
    const PlaySound=()=>{
      if(result_answer.value == "正解！"){
        let music = new Audio(require('../assets/sound/true.mp3'));
        music.play();
      }else{
        let music = new Audio(require('../assets/sound/false.mp3'));
        music.play();
      }
    }

    const load_question=()=>{
      answer1.value = questions_data.value[count.value].ans1;
      answer2.value = questions_data.value[count.value].ans2;
      answer3.value = questions_data.value[count.value].ans3;
      answer4.value = questions_data.value[count.value].ans4;
      r_ans.value = questions_data.value[count.value].r_ans;
      q_sentence.value = questions_data.value[count.value].question;
    }

    const choice_ans=(choice)=>{
      if(choice == r_ans.value){
        result_answer.value = "正解！";
        PlaySound();
        amount_corrects.value++
        console.log("正解",result_answer.value);
      }else{
        result_answer.value = "不正解…";
        PlaySound();
        console.log("不正解",result_answer.value);
      }
    }

    const next_question=()=>{
      count.value++
      load_question();
    }

    return{
      questions_data,
      count,
      amount_corrects,
      result_answer,
      QP_view,
      CP_view,
      answer1,
      answer2,
      answer3,
      answer4,
      r_ans,
      q_sentence,

      load_question,
      PlaySound,
      choice_ans,
      next_question,
    }
  }
}
</script>
<style>
  .question_area{
    border:double 5px;
    width:70%;
    margin:0 auto;
  }

  .question_type{
    text-align:left;
  }

  .ans1{
    border:solid 2px;
    border-radius:10px;
    padding:10px;
    width:150px;
  }
  .ans2{
    margin-left:20px;
    border:solid 2px;
    border-radius:10px;
    padding:10px;
    width:150px;
  }
  .ans3{
    border:solid 2px;
    border-radius:10px;
    padding:10px;
    width:150px;
  }
  .ans4{
    margin-left:20px;
    border:solid 2px;
    border-radius:10px;
    padding:10px;
    width:150px;
  }

  table{
    padding-top:40px;
    margin:0 auto;
  }
</style>