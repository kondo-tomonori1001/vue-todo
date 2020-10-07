<template>
  <div>
    <form>
      インデックスの追加
      <input type="text" v-model="label">
      <button v-on:click.prevent="labelAdd">Add</button>
    </form>
    <hr>
    <form>
      Todoリストの追加
      <input type="text" v-model="comment">
      <button v-on:click.prevent="todoAdd">Add</button>
    </form>
    <div>
      <span>インデックスの選択</span>
      <label v-for="label in labels" v-bind:key="label.id">
        <input type="checkbox" v-bind:value="label.text" v-model="checkLabels">{{ label.text }}
      </label>
    </div>
    <hr>
    <div>
      <label v-for="label in stateOptions" v-bind:key="label.value">
        <input type="radio" v-model="stateFilter" v-bind:value="label.value">{{ label.label }}
      </label>
    </div>
    <div>
      <label v-for="label in labels" v-bind:key="label.id">
          <input type="checkbox" v-bind:value="label.text" v-model="searchLabels">{{ label.text }}
      </label>
    </div>
    <table>
      <thead>
        <tr>
          <th>COMMENT</th>
          <th>STATE</th>
          <th>INDEX</th>
          <th>-</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for = "todo in computedTodos" v-bind:key= "todo.id" v-bind:class="{done:todo.state}">
          <th>{{ todo.comment }}</th>
          <th>
            <button v-on:click="changeState(todo)">{{ stateLabels[todo.state] }}</button>
          </th>
          <th><span v-for="label in todo.labels" v-bind:key="label">{{ label }}</span></th>
          <th>
            <button v-on:click="removeTodo(todo)" >Del</button>
          </th>
        </tr>
      </tbody>
    </table>
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

    // インデックスのフィルター
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
              break
            } else if(item.labels.includes(this.searchLabels[this.searchLabels.length - 1])){
              // 選択されたインデックスの最後が含まれていたら、filterで返す
              return item
            }
          }
        })
      }
    },

    // 作業状態のフィルター機能
    computedTodos:function(){
      return this.indexFilterOr.filter(function(item) {
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
</style>
