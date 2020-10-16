<template>
  <div class="l-main">
    <div class="l-top">
      <div class="l-addIndex">
        <p>インデックスを追加する
        </p>
        <form>
          <input type="text" v-model="label">
          <button v-on:click.prevent="labelAdd">Add</button>
        </form>
      </div>
      <div class="l-addTodo">
        <p>Todoリストを追加する</p>
        <form>
          <input type="text" v-model="comment">
          <button v-on:click.prevent="todoAdd">Add</button>
        </form>
      </div>
    </div>
    <div class="l-container">
      <label class="p-checkbox" v-for="label in labels" v-bind:key="label.id">
        <input type="checkbox" v-bind:value="label.text" v-model="checkLabels"><span>{{ label.text }}</span>
      </label>
    </div>
    <div class="l-resultWrap">
      <div class="l-container">
        <label v-for="label in stateOptions" v-bind:key="label.value">
          <input type="radio" v-model="stateFilter" v-bind:value="label.value"><span>{{ label.label }}</span>
        </label>
      </div>
      <div class="l-container l-container--searchType">
        <label class="p-searchType" v-for="label in searchOptions" v-bind:key="label.value">
          <input type="radio" v-model="searchType" v-bind:value="label.value"><span>{{ label.label }}</span>
        </label>
      </div>
      <div class="l-container">
        <label v-for="label in labels" v-bind:key="label.id">
            <input type="checkbox" v-bind:value="label.text" v-model="searchLabels"><span>{{ label.text }}</span>
        </label>
      </div>
      <table class="p-todoTable">
        <thead>
          <tr>
            <th style="width:80px">状態</th>
            <th>TODO</th>
            <th style="width:250px">タグ</th>
            <th style="width:80px">-</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for = "todo in computedTodos" v-bind:key= "todo.id" v-bind:class="{done:todo.state}">
            <td>
              <button v-on:click="changeState(todo)">{{ stateLabels[todo.state] }}</button>
            </td>
            <td class="u-textLeft">{{ todo.comment }}</td>
            <td class="u-textLeft"><span class="p-tag" v-for="label in todo.labels" v-bind:key="label">{{ label }}</span></td>
            <td>
              <button class="p-delBtn" v-on:click="removeTodo(todo)" >削除</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
  export default{
    // ---------- data ------------
    data:function(){
      return {
        comment:'',
        todos:[],
        stateOptions:[
          {value:-1,label:'すべて'},
          {value:0,label:'作業中'},
          {value:1,label:'完了'}
        ],
        stateFilter:'-1',
        label:"",
        labels:[],
        checkLabels:[],
        searchLabels:[],
        searchType:'or',
        searchOptions:[
          {value:'or',type:"or",label:"ORで絞込み"},
          {value:'and',type:"and",label:"ANDで絞込み"},
        ]
      }
    },

    // ---------- created -----------
    created:function(){
      // ローカルストレージ（以下、LS）に保存されたJSONデータをオブジェクトに変換
      let todos = JSON.parse(localStorage.getItem('todos'));
      // idをセット
      
      // LSが空の場合todosはnullを返す
      // falsyな値であるnullがbooleanコンテキストに現れると、暗黙的にBolleanに型変換されてfalseを返す
      if(todos){
        todos.forEach(function(todo,index){
          todo.id = index;
        })
        this.todos = todos
      } else {
        // LSが空であれば、todosも空にする。
        todos = [];
      }

      let labels = JSON.parse(localStorage.getItem('labels'));
       labels.forEach(function(label,index){
        label.id = index;
      })
      if(labels){
        this.labels = labels
      } else {
        labels = [];
      }
    },

    // ---------- methods -----------
    methods:{
      // todoアイテムの追加
      todoAdd:function(){
        if(this.comment.length === 0){
          return;
        }
        this.todos.push({
          id:this.todos.length + 1,
          comment:this.comment,
          state:0,
          labels:this.checkLabels,
        })
        // テキストボックスを空白にする
        this.comment = "";
        this.checkLabels = [];
      },

      // todoアイテムの削除
      removeTodo:function(item){
        // todosの何番目かを取得
        const index = this.todos.indexOf(item);
        this.todos.splice(index,1);
      },

      // 状態の変更
      changeState:function(item){
        if(item.state === 0){
          item.state = 1 
        } else {
          item.state = 0
        }
        localStorage.setItem('todos',JSON.stringify(this.todos));
      },

      // ラベルの追加
      labelAdd:function(){
        if(this.label.length === 0){
          return;
        }
        this.labels.push({
          id:this.labels.length + 1,
          text:this.label,
        })
        this.label = "";
      }
    },

    // -------- computed ---------
    computed:{
      // [{0:"作業中"},...]に変換
      stateLabels:function(){
        return this.stateOptions.reduce(function(a,b){
          return Object.assign(a,{[b.value]:b.label});
        },{})
      },

    // インデックスのフィルター(Or検索)
    indexFilterOr:function(){
      // どのインデックスにもチェックが無かったら、全てを表示する。
      if( this.searchLabels.length === 0){
        return this.todos;
      } else {
        // インデックスにチェックがあった場合
        return this.todos.filter((item) => {
          for( let i in this.searchLabels){
            for( let j in item.labels){
              // ・チェックされたインデックス（searchLabels）
              // ・todoリストに記録されたインデックス
              // を一つずつ比較して、フィルターをかける
              if(this.searchLabels[i] === item.labels[j]){
                return item
              }
            }
          }
        })
      }
    },

    // インデックスのフィルター（AND検索）
    indexFilterAnd:function(){
      if( this.searchLabels.length === 0){
        return this.todos;
      } else {
        return this.todos.filter((item) => {
          // 選択されたインデックスとtodoリストを一つずつ比較する
          for(let i in this.searchLabels){
            if(item.labels.includes(this.searchLabels[i]) === false){
              // 選択されたインデックスが含まれ無かったら、ループを抜ける
              break;
            } else if(i === String(this.searchLabels.length - 1) ){
              // 選択されたインデックスの最後が含まれていたら、filterで返す
              if(item.labels.includes(this.searchLabels[i]) === true){
                return item;
              }
            }
          }
        })
      }
    },

    // 作業状態のフィルター機能
    computedTodos:function(){
      let searchType;
      if( this.searchType === 'or' ){
        searchType = this.indexFilterOr;
      } else if( this.searchType === 'and'){
        searchType = this.indexFilterAnd;
      }
      return searchType.filter(function(item) {
          if( this.stateFilter < 0){
            return item
          } else if(this.stateFilter === item.state){
            return item
          }
        }, this)
      },
    },

    // -------- watch ---------
    watch:{
      // todosが変更される度に次が実行
      todos:function(){
        // ローカルストレージへの保存
        localStorage.setItem('todos',JSON.stringify(this.todos));
      },
      // labelが変更される度に実行
      labels:function(){
        // ローカルストレージへの保存
        localStorage.setItem('labels',JSON.stringify(this.labels));
      }
    }
  }
</script>

<style>
  body{
    background-color:#99ffcc;
    padding: 0;
    margin: 0;
  }
  .l-top{
    margin-top:30px ;
  }
  .l-top p{
    text-align: center;
    margin: 10px 0;
  }
  .l-top form{
    text-align: center;
    margin: 10px 0;
  }
  .l-top input{
    border: none;
    padding: 5px;
  }
  .l-top button{
    border: none;
    padding: 5px;
    background-color: #006633;
    color: #fff;
    cursor: pointer;
  }
  .l-addTodo{
    margin-top: 30px;
  }
  .l-container{
    display: flex;
    justify-content: center;
  }
  .l-container--searchType{
    margin-top: 20px;
  }
  .l-resultWrap{
    background-color: #ffffff;
    max-width: 1000px;
    margin: 50px auto;
    padding: 30px 20px;
    border-radius: 20px;
  }
  label{
    position: relative;
    display: block;
    cursor: pointer;
  }
  label > input[type="checkbox"],
  label > input[type="radio"]{
    position: absolute;
    opacity: 0;
    display: none;
  }
  label > input[type="checkbox"] + span,
  label > input[type="radio"] + span{
    background-color:#009966;
    opacity: 0.5;
    color: #fff;
    padding: 1px 10px;
    display: block;
    margin: 5px 5px;
    border-radius: 15px;
  }
  label > input[type="checkbox"]:checked + span,
  label > input[type="radio"]:checked + span{
    opacity: 1;
  }
  
  label > input[type="radio"] + span {
    margin: 5px 0;
    border-radius: 0;
  }

  .p-todoTable{
    width: 100%;
    max-width: 1000px;
    margin: 30px auto 0;
    border-collapse: collapse;
    border-spacing: 0;
    border-bottom:2px solid #009966;
  }
  .p-todoTable thead th{
    color: #009966;
    border-bottom:2px solid #009966;
  }
  .p-todoTable tbody td{
    padding: 10px 0;
    text-align: center;
    border-bottom: 1px dotted #009966;
  }

  .p-tag{
    background-color: #009966;
    color: #fff;
    margin: 0 5px;
    padding: 1px 10px;
    border-radius: 15px;
    display: inline-block;
  }

  .u-textLeft{
    text-align: left !important;
  }

  .p-delBtn{
    border: none;
    background-color: red;
    color: #fff;
    cursor: pointer;
    border-radius: 3px;
  }

</style>
