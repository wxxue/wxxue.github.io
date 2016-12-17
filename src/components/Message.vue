<template>
  <div class="hello">
    <div class = 'container'>
      <h3 class = 'toptitle'>留言板</h3>
      <div class = 'maincontent'>
        <input class = 'username padd' type="text" name="username" placeholder="请输入昵称" v-model = 'username'>
        <div class = 'commentBox'>
          <textarea class = 'usercomment padd' name="usercomment" rows="8" cols="40" Maxlength = '300' placeholder="请输入留言内容，不超过300字" v-model='usercomment'></textarea>
          <div class = 'wordsNum'>{{contentlength}}/300</div>
          <button class = 'submitcomment' v-on:click = 'submits'>发布</button>
        </div>
      </div>
      <div class = 'searchLists'>
        <input class = 'searchInput' type="text" placeholder="请输入标签筛选留言，如：用户反馈" v-model = 'searchValue'>
        <button class = 'searchBtn' v-on:click = 'searchItem'>搜索</button>
      </div>
    <!-- 评论列表 -->
      <div class="lists">
        <div class = 'listsItem' v-for = 'item in lists' :key = 'item.id'>
          <strong>{{item.username}}</strong>：{{item.content}}
          <button class = 'delete' type="button" name="button" :data-id = 'item.id' v-on:click = 'deleteTip'>删除</button>
          <div class="tips">
            <span class = 'tipsItem' v-for = 'tip in item.contentclass' :key = 'item'>{{tip}}<b class = 'reset' :data-id = 'item.id' :data-tip = 'tip' v-on:click = 'resettip'>删除</b></span>
            <span class = 'addtip' v-on:click = 'addtip' :data-id = 'item.id'>添加</span>
            <div class = 'Popup' v-show = 'item.showadd'>
              <input type="text" name="name" value="" v-model = 'addTipInput'>
              <button type="button" name="button" v-on:click = 'cancleBtn' :data-id = 'item.id'>取消</button>
              <button type="button" name="button" v-on:click = 'sureBtn' :data-id = 'item.id'>确定</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      username: '',  //用户昵称
      usercomment: '', //用户输入的留言
      contentlength: 0,  //留言的长度
      addTipInput: '',  //添加标签的内容
      searchValue: '',  //搜索的内容
      lists: [],       //留言列表
      cacheLists: []   //缓存列表（页面中）
    }
  },
  created() {
    var lists = JSON.parse( localStorage.getItem('lists') ),
        newLists,
        newArr;
    // 判断一下从本地存储中取到的lists是不是一个数组
    if( {}.toString.call( lists ).slice( 8, -1 ) != 'Array' ){
      lists = [];
      newLists = [];
      newArr = [];
    }else {
      newLists = lists;
      newArr = this.lists;
    }
    for( var i= 0; i< newLists.length;i++){
      newArr.push( JSON.parse( newLists[i] ) );
    }
    this.lists = newArr;
    // 将数据缓存到一个数组中，用做后期的搜索
    this.cacheLists = this.lists;
  },
  computed: {
    // 计算属性：实时获取到输入框中字符的长度
    contentlength: function() {
      return this.usercomment.length;
    }
  },
  methods: {
    submits() {
      var obj = {
            id: Date.now(),
            username: this.username,
            content: this.usercomment,
            contentclass: ['天天向上', '好好学习'],
            showadd: false
          },
          lists =  JSON.parse( localStorage.getItem( 'lists' ) ),
          newStr = JSON.stringify( obj );
      this.lists.push( obj );
      // 将数据缓存到一个数组中，用做后期的搜索
      this.cacheLists = this.lists;
      lists.push( newStr );
      var newlists = JSON.stringify( lists );
      localStorage.setItem('lists', newlists );
      // 点击提交，数据处理完之后，让输入框清零
      this.username = '';
      this.usercomment = '';
    },
    deleteTip( e ) {
      var objId = e.target.dataset.id,
          newArr = [];
      this.lists.forEach(( el, i )=>{
        if( el.id != objId ){
          newArr.push( el );
        }
      });
      this.lists = newArr;
      // 将数据缓存到一个数组中，用做后期的搜索
      this.cacheLists = this.lists;
      mystorage( this.lists );
    },
    resettip( e ) {
      var objId = e.target.dataset.id,
          objTip = e.target.dataset.tip;
      console.log( this.lists )
      this.lists.forEach( ( item, i )=>{
          if( item.id == objId ) {
            var newTips = [];
            item.contentclass.forEach( ( item, i )=>{
              if( item != objTip ){
                // 将不需要删除的tip保存到新数组中，删除的就直接跳过
                newTips.push( item );
              }
            } );
            // 将新的tip数组赋值给contentclass
            item.contentclass = newTips;
          }
      } );
      // 将数据缓存到一个数组中，用做后期的搜索
      this.cacheLists = this.lists;
      mystorage( this.lists );
    },
    searchItem() {
      console.log( this.searchValue )
      // 获取到搜索框中的数据
      var value = this.searchValue,
          lists = this.cacheLists,
          newArr = [];
          console.log( this.cacheLists );
      lists.forEach( ( el, i )=>{
          el.contentclass.forEach( ( item, index )=>{
            if( item.indexOf( value ) != -1 ){
              newArr.push( el );
            }
          } );
      } );
      this.lists = newArr;
    },
    addtip( e ) {
      var objId = e.target.dataset.id;
      this.lists.forEach( ( item, i )=>{
          if( item.id == objId ) {
            // 让点击添加的一栏显示一个输入框
            item.showadd = true;
          }
      } );
      // 将数据缓存到一个数组中，用做后期的搜索
      this.cacheLists = this.lists;
    },
    cancleBtn( e ) {
      var objId = e.target.dataset.id;
      this.lists.forEach( ( item, i )=>{
          if( item.id == objId ) {
            // 点击取消的时候，让输入框隐藏
            item.showadd = false;
          }
      } );
      // 将数据缓存到一个数组中，用做后期的搜索
      this.cacheLists = this.lists;
    },
    sureBtn( e ) {
      var objId = e.target.dataset.id,
          newtip = this.addTipInput;
      console.log( this.addTipInput )
      this.lists.forEach( ( item, i )=>{
          if( item.id == objId ) {
            item.contentclass.push( newtip );
            // 点击确定的时候，让输入框隐藏
            item.showadd = false;
          }
      } );
      mystorage( this.lists );
      // 将数据缓存到一个数组中，用做后期的搜索
      this.cacheLists = this.lists;
    }
  }
}

// 存储数据的函数
function mystorage( lists ){
  var newArr = [];
  console.log( 'ss' )
  lists.forEach( ( el, i )=>{
    newArr.push( JSON.stringify( el ) );
  } )
  var newlists = JSON.stringify( newArr );
  localStorage.setItem('lists', newlists );
}




</script>


<style scoped>
    .hello{
      display: flex;
    }
    .container {
      padding: 20px 400px;
      box-sizing: border-box;
    }
    .container .toptitle {
      font-size: 30px;
      text-align: center;
    }
    .padd {
      box-sizing: border-box;
      padding: 10px;
    }
    .username {
      display: block;
      width: 230px;
      height: 35px;
      font-size: 15px;
      border-radius: 7px;
      outline-style: none;
      border: 1px solid #ccc;
      margin: 20px 0;
    }
  
    .commentBox {
      position: relative;
     

    }
    .usercomment {
      width: 400px;
      height: 180px;
      font-size: 16px;
      border-radius: 7px;
      border: 1px solid #ccc;
    
    }
    .wordsNum {
      position: absolute;
      left: 15px;
      bottom: 15px;
    }
    .submitcomment {
      width: 57px;
      height: 31px;
      color: #666;
      border: 1px solid #ccc;
      position: absolute;
      right: -85px;
      bottom: 7px;
      outline-style: none;
      background-color: rgba(125, 92, 95, 0.08);
      border-radius: 5px;
      cursor: pointer;
    }
    .submitcomment:hover {
      box-shadow: 0 0 10px #ccc;
    }
    /*搜索框的样式*/
    .searchLists {
      padding: 20px 0;
      position: relative;
    }
    .searchInput {
      width: 390px;
      height: 36px;
      outline-style: none;
      border-radius: 5px;
      box-sizing: border-box;
      padding: 0 40px 0 10px;
    }
    .searchBtn {
      width: 40px;
      height: 36px;
      color: #666;
      position: absolute;
      top: 20px;
      right: -70px;
      border-radius: 5px;
      outline-style: none;
      border: 0 none;
      cursor: pointer;
      background-color: rgba(125, 92, 95, 0.08);
    }
    h1 {
      color: #42b983;
    }
    .lists {
      width: 400px;
      padding: 10px 10px;
      box-sizing: border-box;
    }
    .listsItem {
      /*height: 60px;*/
      margin-bottom: 20px;
      border-bottom: 1px solid #ccc;
      position: relative;
    }
    .delete {
      width: 40px;
      height: 25px;
      color: #666;
      outline-style: none;
      border-radius: 5px;
      border: 1px solid #ccc;
      background-color: rgba(125, 92, 95, 0.08);
      position: absolute;
      top: 11px;
      right: 5px;
    }
    .delete:hover {
      box-shadow: 0 0 10px #ccc;
    }
    .tips {
      padding: 10px 5px;
    }
    .tipsItem {
      width: 40px;
      color: #666;
      font-size: 12px;
      line-height: 30px;
      text-align: center;
      border:1px solid #ccc;
      padding: 5px 35px 5px 5px;
      border-radius: 5px;
      margin: 0 10px;
      position: relative;
    }
    .reset {
      width: 29px;
      line-height: 22px;
      text-align: center;
      position: absolute;
      right: 2px;
      top: 0;
      border-left: 1px solid #ccc;
    }
    .reset:hover {
      cursor: pointer;
      box-shadow: 0 0 10px #ccc;
    }
    .addtip {
      width: 29px;
      font-size: 12px;
      display: inline-block;
      line-height: 22px;
      text-align: center;
      border: 1px solid #ccc;
      border-radius: 5px;
      cursor: pointer;
      box-shadow: 0 0 10px #ccc;
    }
    .addtip:hover {
      box-shadow: 0 0 25px #ccc;
    }
</style>
