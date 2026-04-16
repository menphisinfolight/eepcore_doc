# Server 端常見問題與技巧（FAQ）

> 來源：infolight.com 官方常見問答（EEP.NET Core / Server 分類）
> 整理日期：2026-04-16
> 共 26 題，依主題分類整理

---

## SQL與資料操作

### [#01] 如何在前端下SQL語法取得資料?

基於資訊安全的原因，      .net Core      是不允許在前端直接執行      SQL      語法的，只能夠透過後端的      Server Method      來執行。
後端的      ServerMethod      寫法，如下      :

```csharp
        public object getsql (dynamic objParam)
        {
           string id = objParam["id "]; //      取得前端傳至後端的參數      "id "
           string sql= "select * from       員工資料表      where       員工編號      ='"+id+"'"; //SQL      條件句
           DataTable Tmp = ExecuteDataTable("EEPCore ", sql); //      執行      SQL      ，      "EEPCore "      為執行的資料庫
```

int count = Tmp.Rows.Count;
string retval = "";
object ret;

```csharp
           if(count >0){
```

retval = Tmp.Rows[0]["      姓名      "].ToString();

```csharp
               ret = new object[] {retval};
           }
           else {
               ret = new object[] {""};
           }
           return ret;
        }

```

前端的調用，如下的      JS

```csharp
        function Get_Information()
        {
           var result = $.callSyncMethod('      員工資料表      2 ', 'getsql ',{id:'001 '}); //      非同步調用
           var param = $.parseJSON(result); //      傳回      result      為      JSON      格式
           if (param.length != 0){
               alert(param[0]); //      假設只傳回      1      筆資料
          }
        }
        //      員工資料表      2 ->      後端模組名稱
        //getsql ->      後端      ServerMethod       名稱
        //id:'001 '->      傳遞給後端的參數名稱      &      值
```

---

### [#02] 如何在前端執行後端的SQL?

舉例                :
去後端更改某欄位的內容，前端的      JS:

```csharp
        function Do_ServerMethod(){
           $.callMethod('      員工資料表      2 ','dosql ',{id:'001 '},function(result){
            var msg = $.parseJSON(result); //取得回傳字串
```

alert(      msg      );

```csharp
           });
        }
        //      員工資料表      2 ->      後端模組名稱
        //dosql ->      後端      ServerMethod       名稱
        //id:'001 '->      傳遞給後端的參數名稱      &      值


```

後端      servermethod:

```csharp
        public string dosql(dynamic param)
        {
               string id = param["id "];//      取得前端傳至後端的參數      "id "
           try{
```

string sql= "update       員工資料表      set       姓名      ='test 'where       員工編號      ='"+id+"'";

```csharp
               ExecuteNonQuery("EEPCore ", sql); //      執行      SQL      ，      "EEPCore "      為執行的資料庫
               return "      執行成功      ";
           }
           catch (Exception e){
               return "      執行失敗      ";
           }
        }
```

---

### [#03] 如何在新增或修改後去執行一段SQL?

可以在      Server      端的      Updatecomponent      的      OnAfterApplied       雙擊產生一個      public
並在裡面處理      ,      舉例如下

```csharp
        public void uc      員工資料表      _onAfterApplied(object sender, dynamic rows, List <string >sqls)
        {
           var id = "";
```

string sql = "";

```csharp
           if(rows.deleted.Count == 0 ){
               try{
                   if(rows.updated.Count >0) {//      判斷是否為      修改
```

id = rows.updated[0].      員工編號      ;
sql = "update       客戶資料表      set       負責業務名稱      ='update 'where       負責業務      ='"+id+"'";

```csharp
                   }
                   else if(rows.inserted.Count >0) {//      判斷是否為      新增
```

id = rows.inserted[0].      員工編號      ;
sql = "update       客戶資料表      set       負責業務名稱      ='insert 'where       負責業務      ='"+id+"'";

```csharp
                   }
                   if(id != ""){
                       ExecuteNonQuery("EEPCore ", sql); //      執行      SQL      ，      "EEPCore "      為執行的資料庫
                   }
               }
               catch(Exception e){
                   throw new Exception(e.Message);
               }
           }
        }
```

---

### [#04] 如何Call Stored 取得資料?

舉例如下      ,
前端      JS

```csharp
        function Call_SP(){
           var result = $.callSyncMethod('      員工資料表      2 ', 'callsp ',{id:'1 '}); //      非同步調用
           var param = $.parseJSON(result); //      傳回      result      為      JSON      格式
           if (param.length != 0){
               alert(param[0]); //      假設只傳回      1      筆資料
          }
        }

```

後端的      ServerMethod

```csharp
        public object callsp(dynamic objParam)
        {
           string id = objParam["id "]; //      取得前端傳至後端的參數
           string sql= "exec       客戶資料表      _sp '"+id+"'"; //SQL      條件句
           DataTable Tmp = ExecuteDataTable("EEPCore ", sql); //      執行      SQL      ，      "EEPCore "      為執行的資料庫
```

int count = Tmp.Rows.Count;
string retval = "";
object ret;

```csharp
           if(count >0){
```

retval = Tmp.Rows[0]["      名稱      "].ToString();

```csharp
               ret = new object[] {retval};
           }
           else {
               ret = new object[] {""};
           }
           return ret;
        }
```

---

### [#14] 如何從後端撈取資料後，load資料到前端的datagrid上?

範例參考如下:
RWD前端程式碼:

```csharp
  function test()
  {
    $.callMethod('出貨單 ', 'testServerMethod ', {OrderID: 'Q1912001 '}, function(result){
      var rows = $.parseJSON(result); //將JSon轉回Object類型提供給Grid顯示
      $('#dgDetail ').datagrid('loadData ', {rows:rows,total:rows.length});
    })
  }


```

Server端程式碼:

```csharp
  public object testServerMethod(dynamic objParam)
  {
    string OrderID = objParam["OrderID "]; //取得前端傳至後端的參數
    string sql= "select * from 報價單明細 where 報價單號 = '"+ OrderID +"'"; //SQL條件句
    var table = ExecuteDataTable(ClientInfo.Database, sql);
    return table;
  }
```

---

### [#15] 如果在後端執行多句ExecuteNonQuery，是否可以將其合併讓畫面更簡潔?

可以的，範例參考如下:

```csharp
  string sql= "UPDATE 客戶資料表 SET 負責人=N 'APPLE 'WHERE 客戶編號='1 '";
  string sql2= "UPDATE 客戶資料表 SET 負責人=N 'BANANA 'WHERE 客戶編號='2 '";
   ExecuteNonQuery("ERPS ", new string[]{ sql, sql2});
```

---

### [#22] 我希望 Server 端的 SQL 語句執行時間不要太久，並且使用參數化的方式傳入 WHERE 條件，請問 serverMethod 應該怎麼寫比較好?

參考如下                :

```csharp
public string test(dynamic objParam)
        {
           try{
               var parameters = new Dictionary <string, object >
               {
                   { "orderid ", "001 "}
               };
               ExecuteDataTable(ClientInfo.Database, "Select * from Orders where OrderID = @orderid ", parameters, 30);
               //30      表示      commantTimeout      時間，單位為秒
               return "ok ";
           }
           catch(Exception e){
               return e.Message;
           }
        }
```

---

### [#23] 請問透過 ExecuteDataTable 方法取出資料後，能否將資料的完整內容直接回傳至前端進行後續處理?

可以的，參考如下      :

```csharp
        public object callmethod1(dynamic param) {
           var table = ExecuteDataTable(ClientInfo.Database, "Select * from Orders ");
           return Newtonsoft.Json.JsonConvert.SerializeObject(table);
                 //      將      Select      出來的資料直接完整回傳
        }
```

---

## 存檔前後處理

### [#05] 如何在Server端存檔前(新增或更改) 再決定某個欄位的內容並存入資料庫?

舉例如下      ,
後端      servermethod

```csharp
        public bool uc      報價單      _onBeforeInsert(object sender, dynamic row, List <string >sqls)
        {
           string sql= "select * from       客戶資料表      where       客戶編號      ='"+ row["      客戶編號      "] + "'"; //SQL      條件句
           DataTable Tmp = ExecuteDataTable("EEPCore ", sql); //      執行      SQL      ，      "EEPCore "      為執行的資料庫
```

int count = Tmp.Rows.Count;
string retval = "";

```csharp
           if(count >0){
               row["      名稱      "] = Tmp.Rows[0]["      名稱      "].ToString();//      改變某欄位的內容
           }
           else {
               row["      名稱      "] = "";//      改變某欄位的內容
           }

               return true;
        }
```

---

### [#16] 如何在更新資料前，取得該筆資料更改前的欄位值?

可以在Server端的updateComponent，在onBeforeUpdate的時機點透過以下語法取得:

```csharp
  var oldRow = this.GetComponent <UpdateComponent >("uc員工資料表 ").GetCommand().GetOldRow(row);
   var oldName = oldRow["姓名 "].ToString();
```

---

### [#19] 請問Server端UpdateComponent的onBeforeUpdate時機點，可以取得當前這筆資料更改前的舊值嗎?

可以的，請在onBeforeUpdate時機點使用以下程式碼取得更新前的資料

```csharp
  var oldRow = this.GetComponent <UpdateComponent >("uc員工資料表 ").GetCommand().GetOldRow(row);
   var name = oldRow["姓名 "].ToString();
        //uc員工資料表請替換為UpdateComponent的ID
```

---

### [#24] 當前端透過importExcel方法上傳資料後，我能否在後端，動態的控制其中某一個欄位的內容?

前端      :

```csharp
        function importExcel2(){
           $('#dgMaster ').datagrid('importExcel ', {
```

beforeImport: '      產品資料表      .BeforeImport '

```csharp
               //      產品資料表為      Server      端名稱，      BeforeImport      為其中的      Method      名稱
           });
        }
```

後端      :

```csharp
        public void BeforeImport(JObject row, List <string >sqls)
        {
```

row["      產品說明      "] = "abc ";

```csharp
           //      將      "      產品說明      "      欄位內容替換為      "abc "
           //Excel      上的每一筆資料都會進入此      Method
        }
```

---

## Server元件設定

### [#08] 後端Command中的SQL比較複雜, 用了很多Left Join與欄位AS別名後, 前端的Query送出後會發生Where錯誤, 因為前端並不知道後端的實際的欄名, 如何解決?

可以透過後端      Command      的      onBeforeExecuteSQL      事件來重新替換      Where      的欄位名稱      ,       如下的範例      :

```csharp
        public string       客戶資料表      _onBeforeExecuteSQL(object sender, string sql, List <string >whereStrs)
        {
           for (var i = 0; i <whereStrs.Count; i++) {
               whereStrs[i] = whereStrs[i].Replace("M_SVS ", "A.M_MA "); //      將      M_SVS      別名欄位換成實際的      A.M_MA      欄位
           }
           return sql;
        }
```

---

### [#09] 為何我在 Server端 InfoCommand中使用Group By命令, 沒Select到Key欄位時, 會報錯 "Group By子句"的錯誤?

這是因為      InfoCommand      預設      SelectPaging=True      的屬性      (      資料快速分頁功能      ),       如果要特殊使用      Group by      時      ,       須配合      SelectPaging=False      即可。

---

### [#10] 如何正確使用Server端infocommand的SecStyle和SecExcept屬性?

SecExcept      要配合      SecStyle, SecStyle=User      那      SecExcept      要填      Userid,
如果      SecStyle=Group, SecExcept      則填      Groupid,
如果      SecExcept      要填多個以”      ,      ”或”      ;      ”隔開

---

### [#11] 自動編號中, 如何使用指定欄位作為前引碼來編號?

舉例如下      ,

```csharp
        public string an      報價單      _onGetFixed(object sender, string fixedString, dynamic rows)
        {
```

fixedString = rows[0].      名稱      ;

```csharp
               return fixedString;
        }
```

---

### [#17] 如何在每次撈取資料時，都添加額外的where條件?

可以在InfoCommand的onBeforeExcuteSQL屬性添加，
範例如下:

```csharp
  public string 員工資料表_onBeforeExecuteSQL(object sender, string sql, List <string >whereStrs)
  {
```

whereStrs.Add("員工編號 = '001 '");

```csharp
        	      return sql;
  }
```

---

## 進階開發

### [#06] 如何在 .NetCore中開發Stored Procedure?

在      .Net Core      的      Table      節點中      ,       可以右鍵      "      儲存過程      "(      預存程序      ),       打開頁面後可以用      "+      新增      "      來增加一個      Stored Procedure,       如下      ,       填寫                Name            為      Stored Procedure Name, Parameters      為      Input/Output      參數      , Text      代表      Stored Procedure      的程式內容，並按下確定即可。

---

### [#07] TRS交易時, 是否可以控制某些條件成立在過帳, 否則就不過帳?

可以的      ,       在      Server      的      Transaction      組件中      ,       每一個交易都有一個      onBeforeTrans      事件      ,       可以控制是否要持續交易或動態改變對方的交易欄位      ,       如下為控制是否交易的例子      :

```csharp
        public bool InfoTransaction1_onBeforeTrans(object sender, dynamic row, dynamic oldRow, List <InfoTransaction.Field >fields)
        {
           if(row["      請假類型      "] != '      特休假      ') return false;//      不為特休假      就不過帳
               return true;//      正常過帳
        }

```

上面的情況是針對      '      請假類型      '      不能更改的情況      ,       如果請假類型可以更改的話      ,       要改成這樣      :

```csharp
        public bool InfoTransaction1_onBeforeTrans(object sender, dynamic row, dynamic oldRow, List <InfoTransaction.Field >fields)
        {
           if (row["      請假類型      "] !='      特休假      '&&oldRow["      請假類型      "]!='      特休假      ') { //      考慮更改時新舊值都不為特休假
            return false;
          }
          else if (oldRow["      請假類型      "] == '      特休假      ') { //      如果是特休假改為其他假時
```

row["      請假天數      "] = 0;

```csharp
          }
          return true;
        }

        // oldRow ["      欄位名稱      "] ->      修改前      的欄位內容
        // row["      欄位名稱      "]    ->      修改後      的欄位內容
```

---

### [#12] 如何發信?

舉例如下      ,
前端      js

```csharp
        function Send_Mail(){
           $.callMethod('      員工資料表      2 ','sendMail ',{},function(result){
```

alert('      發信成功      !');

```csharp
           });
        }

```

後端
先拉一個      infomail      組件到      server      端設計畫面中
然後再貼上以下的範例      ,      並自行調整

```csharp
        public object sendMail(dynamic objParam)
        {
           var mail = this.GetComponent <InfoMail >("InfoMail1 ");//infoMail      元件      ID
```

mail.Subject = "test ";
mail.Body = "test <br >123 ";
mail.To = "aaa@gmail.com ";
mail.Send();

```csharp
        return "";
        }
```

---

### [#13] 如果想要在Server端的C#代碼，添加額外的using參考時應該如何調整?

```csharp
請調整      EEPGlobal.Core      下的      MSBuild.cs      ，添加需要的      using      參考後重      build
```

---

### [#18] 如何透過後端ServerMethod起單?

舉例如下:

```csharp
        public object DoApprove(dynamic param)
        {
```

string SQL = "";

```csharp
          //取得選取資料
          SQL = "SELECT * FROM 出貨申請單 WHERE 出貨單號= 'S202301001 '";
          DataTable dtMain = ExecuteDataTable(ClientInfo.Database, SQL);
          this.ClientInfo.User = '001 '; //起單的用戶ID
          if (dtMain.Rows.Count >0)
          {
            var row = dtMain.Rows[0].ToJSON();
            return SubmitFlow(row, new JObject
                     {
                       { "method ", "Start "}, //Start 代表起單且送往流程第一關 ,Prepare 代表起單後暫存在起單人的待辦
                       { "FlowID ", "出貨申請流程 "},  //Flow名稱
                       { "Role ", "R01 "},  //起單人的角色ID
                       { "WEBFORM_NAME ", "出貨單 "}, //表單名稱
                       { "tabTitle ", "出貨單 "},   //頁籤名稱
                       { "FORM_KEYS ", "出貨單號 "},  //key
                       { "FORM_PRESENTATION ", "出貨單號:'S202301001 '"},
                       { "FORM_PRESENTATION_CT ", "出貨單號:'S202301001 '"},
                       { "PROVIDER_NAME ", "s出貨單.出貨單 "}, //資料來源remoteName
                        { "url ", "http://localhost:5050/"} //runtime網址
                     });
          }
          else{
            return "無尚未起單之單據!";
          }


        }
```

---

### [#20] 請問我想要從A Server端，呼叫B Server端的serverMethod，是否有方法可以達成?

請參考以下方式:

```csharp
        new DataModule(ClientInfo, "Server端名稱 ").CallMethod("method名稱 ",new JObject{});
```

---

### [#21] 請問Oracle資料庫環境下，如何執行Store Procedure並取得回傳值?


以下為Server端透過ORACLE PACKAGE.SP取得資料集合:
1.
以下為我的PACKAGE下的Body SP內容，
有一傳入參數ID，DATA_cursor為 Oracle的SYS_REFCUSOR指標
2.
EEPGlobal.Core/MSBuild.cs添加以下兩個引用參考，
重建建置後，Server端的C#即可使用參考下的方法
Oracle.ManagedDataAccess.Client
Oracle.ManagedDataAccess.Types
3.
後端C#參考如下

```csharp
  public object GetSPData(dynamic objParam)
  {
    DataTable dataTable = new DataTable();
    using (var db = DatabaseHelper.Create(ClientInfo, ClientInfo.Database, DatabaseType.Normal))
    {
      var conn = db.Connection;
      var cmd = conn.CreateCommand() as OracleCommand;
```

cmd.BindByName = true;
cmd.CommandType = System.Data.CommandType.StoredProcedure;

```csharp
      //呼叫MYPACKAGE下的GETDATA預存程序
```

cmd.CommandText = "MYPACKAGE.GETDATA ";

```csharp
      // 輸入參數
      var inParam = cmd.CreateParameter() as OracleParameter;
```

inParam.ParameterName = "ID ";
inParam.Value = "1 ";
inParam.DbType = DbType.String;
inParam.Direction = ParameterDirection.Input;

```csharp
      // 輸出參數
      var outParam = cmd.CreateParameter() as OracleParameter;
```

outParam.ParameterName = "DATA_cursor ";
outParam.OracleDbType = OracleDbType.RefCursor;
outParam.Direction = ParameterDirection.Output;
cmd.Parameters.Add(inParam);
cmd.Parameters.Add(outParam);

```csharp
      cmd.ExecuteNonQuery();
      // 讀取結果集
      if (outParam.Value is OracleRefCursor)
      {
```

OracleRefCursor refCursor = (OracleRefCursor)outParam.Value;

```csharp
        using (OracleDataReader dr = refCursor.GetDataReader())
        {
          // 使用 DataTable.Load 方法將資料集合存放於 DataTable 中
```

dataTable.Load(dr);

```csharp
        }
      }
    }
    return Newtonsoft.Json.JsonConvert.SerializeObject(dataTable);
  }


```

4.
回傳結果如下:

---

### [#25] 當我的Stroed Procedure透過資料庫工具執行撈取的速度非常快，透過系統的infoCommand或是serverMethod呼叫卻慢了許多，請問可能是甚麼原因?

出現此情況可能和      Parameter Sniffing      有關，
當你第一次執行儲存過程時，      SQL Server       會根據當時傳入的參數，產生並快取一份執行計劃。
如果你第一次用的是      %      ，      SQL Server       會產生適合「查全部資料」的計劃，讓查少量資料時也變得很慢。
因此      Stored Procedure      內的      Select .... Where Column Like ...       這樣的命令之後要加上
OPTION (RECOMPILE)

---

### [#26] 在serverMethod內撰寫的程式碼，是否有簡單的方式可以幫我紀錄log?

可以的，

```csharp
          new LogHelper(ClientInfo).Log(LogHelper.LogType.Normal, LogHelper.LogStyle.UserDefine, "title ", "message ", DateTime.Now, 0);
```

當      server      端的      method      內加上這一行後，
執行到該行，會自動紀錄      Log      到系統的日誌內，日誌類型會為      UserDefine      ，
只需要動態控制這筆日誌的標題      (title)      和備註      (message)      即可。
[圖片]

---
