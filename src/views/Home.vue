<template>
  <div class="home">
    <!-- タイトル画面 -->
    <div class="title" :style="{display:title_view}">
      <h1>なんでもクイズ</h1>
      <button @click="start_quiz">スタート</button>
      <button @click="select_question">問題集選択</button>
    </div>

    <!-- 問題選択画面 -->
    <div class="select_questions" :style="{display:selecter_view}">
      <input type="file" @change="fileChange" id="file_input_expense">
      <button @click="downloadCSV">テンプレートDL</button>
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
    ])

    let amount_corrects = ref()

    let result_answer = ref()
    
    // 問題数カウント
    let count = ref(0)

    // 画面表示管理
    let title_view = ref("block")
    let selecter_view = ref("none")
    let QP_view = ref("none")
    let CP_view = ref("none")

    // 選択肢・正解・問題文
    let answer1 = ref()
    let answer2 = ref()
    let answer3 = ref()
    let answer4 = ref()
    let r_ans = ref()
    let q_sentence = ref()

    // 音源再生
    const PlaySound_TF=()=>{
      if(result_answer.value == "正解！"){
        let music = new Audio(require('../assets/sound/true.mp3'));
        music.play();
      }else{
        let music = new Audio(require('../assets/sound/false.mp3'));
        music.play();
      }
    }

    const start_quiz=()=>{
      change_view(title_view,"none","block")
      change_view(QP_view,"none","block")
      selecter_view.value = "none"
      load_question()
    }

    const select_question=()=>{
      change_view(selecter_view, "none", "block")
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
        change_view(QP_view, "none", "block");
        change_view(CP_view, "none", "block");
        PlaySound_TF();
        amount_corrects.value++
      }else{
        result_answer.value = "不正解…";
        change_view(QP_view, "none", "block");
        change_view(CP_view, "none", "block");
        PlaySound_TF();
      }
    }

    const change_view=(view, dis1, dis2)=>{
      if(view.value == dis1){
        view.value = dis2;
      }else{
        view.value = dis1;
      }
    }

    const finish_quiz=()=>{
      console.log("クイズ終了");
    }

    const next_question=()=>{
      if(count.value < questions_data.value.length-1){
        count.value++
        load_question();
        change_view(QP_view, "none", "block");
        change_view(CP_view, "none", "block");
      }else{
        finish_quiz();
      }
    }

    // CSV関連
    const json2csv=(json)=> {
      var header = Object.keys(json[0]).join(',') + "\n";

      var body = json.map(function(d){
          return Object.keys(d).map(function(key) {
              return d[key];
          }).join(',');
      }).join("\n");

      return header + body;
    }

    const getDate=(dateObj)=>{
      let now_date = dateObj.getFullYear() + '年' + 
      ('00' + (dateObj.getMonth() + 1)).slice(-2) + '月' + 
      ('00' + dateObj.getDate()).slice(-2) + '日' + 
      ('00' + dateObj.getHours()).slice(-2) + '時' + 
      ('00' + dateObj.getMinutes()).slice(-2) + '分';
      return now_date;
    }

    const downloadCSV=()=> {
        //ダウンロードするCSVファイル名を指定する
        let dateObj = new Date()
        let now_date = getDate(dateObj);
        const filename = now_date+".csv";
        //CSVデータ
        const data = json2csv(questions_data.value);
        //BOMを付与する（Excelでの文字化け対策）
        const bom = new Uint8Array([0xef, 0xbb, 0xbf]);
        //Blobでデータを作成する
        const blob = new Blob([bom, data], { type: "text/csv" });

        //IE10/11用(download属性が機能しないためmsSaveBlobを使用）
        if (window.navigator.msSaveBlob) {
            window.navigator.msSaveBlob(blob, filename);
        //その他ブラウザ
        } else {
            //BlobからオブジェクトURLを作成する
            const url = (window.URL || window.webkitURL).createObjectURL(blob);
            //ダウンロード用にリンクを作成する
            const download = document.createElement("a");
            //リンク先に上記で生成したURLを指定する
            download.href = url;
            //download属性にファイル名を指定する
            download.download = filename;
            //作成したリンクをクリックしてダウンロードを実行する
            download.click();
            //createObjectURLで作成したオブジェクトURLを開放する
            (window.URL || window.webkitURL).revokeObjectURL(url);
        }
    }

    const fileChange=(e)=> {
      const file = e.target.files[0];
      const reader = new FileReader();
      reader.readAsText(file, 'Shift_JIS');
      // const CSVs = [];

      const loadFunc = () => {
        const lines = reader.result.split("\n");
        lines.forEach((element, index) => {
          if( index === 0 ){ return; }
          const CSV_data = element.split(",");
          // if (CSV_data.length != 3) return;
          const CSV = {
            question: CSV_data[0],
            ans1: CSV_data[1],
            ans2: CSV_data[2],
            ans3: CSV_data[3],
            ans4: CSV_data[4],
            r_ans: CSV_data[5],
          };

          questions_data.value.push(CSV);
          // item_list.value = item_list.value.shift;
          console.log(questions_data.value[0])
          // item_list.value.delete(1,3)
          console.log("CSV",CSV)
        });
        // this.CSVs = CSVs;
      };
      reader.onload = loadFunc;
      reader.readAsBinaryString(file);
      console.log("問題データ:",questions_data.value)
    }


    const fileChange_preset=(e)=> {
      const file = e.target.files[0];
      const reader = new FileReader();
      reader.readAsText(file, 'Shift_JIS');
      // const CSVs = [];

      const loadFunc = () => {
        const lines = reader.result.split("\n");
        lines.forEach((element, index) => {
          if( index === 0 ){ return; }
          const CSV_data = element.split(",");
          // if (CSV_data.length != 3) return;
          const CSV = {
            name: CSV_data[0],
            price: CSV_data[1],
            remarks: CSV_data[2],
            amount: CSV_data[3]
          };

          questions_data.value.push(CSV);
          // item_list.value = item_list.value.shift;
          console.log(questions_data.value[0])
          // item_list.value.delete(1,3)
          console.log("CSV",CSV)
        });
        // this.CSVs = CSVs;
      };
      reader.onload = loadFunc;
      reader.readAsBinaryString(file);
      console.log("問題データ:",questions_data.value)
    }


    return{
      questions_data,
      count,
      amount_corrects,
      result_answer,
      title_view,
      selecter_view,
      QP_view,
      CP_view,
      answer1,
      answer2,
      answer3,
      answer4,
      r_ans,
      q_sentence,

      start_quiz,
      select_question,
      load_question,
      PlaySound_TF,
      choice_ans,
      next_question,
      change_view,

      json2csv,
      downloadCSV,
      getDate,
      fileChange,
      fileChange_preset,
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