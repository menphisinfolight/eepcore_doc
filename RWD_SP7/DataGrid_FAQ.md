# DataGrid 常見問題與技巧（FAQ）

> 來源：infolight.com 官方常見問答（EEP.NET Core / DataGrid 分類）
> 整理日期：2026-04-16
> 共 93 題，依主題分類整理

---

## 欄位取值與設值

### [#643] 如何取得DataGrid欄位的值?

```javascript
var rows = $('#dgMaster').datagrid('getRows');
 var index=$('#dgMaster').datagrid('getSelectedIndex');
 var row=rows[index];
 alert(row.欄位名稱); //欄位名稱請自行替換
```

---

### [#678] 請問,如何用JS去更改DataGrid的欄位內容?

如下,透過DataGrid的getRows與updateRow的方法:

```javascript
 function updTax() //更新稅額
 {
   var rows = $('#dgDetail').datagrid('getRows');
 for(var i = 0; i <rows.length; i ++){
  var row = rows[i];
     if(row.類別=='代扣所得' ){
       var amt= -($('#dptab_薪資設定_所得扣繳金額').val());
    $('#dgDetail').datagrid('updateRow',{index: i,row: {'金額': amt}});
  }
     if(row.類別=='代扣保險' ){
       var amt= -($('#dptab_勞保健保資料_本人勞健保費').val());
    $('#dgDetail').datagrid('updateRow',{index: i,row: {'金額': amt}});
  }
    }
 }
```

---

### [#734] DataGrid如何取得欄位加總值?

JQ取值方法:

```javascript
   var footer =$('#dgMaster').datagrid('getFooterRows');
   alert(footer[0].欄位);
```

RWD取值方法:

```javascript
   var totalRow = $('#dgMaster').datagrid('getTotal');
   alert(totalRow.欄位);
```

---

### [#752] 如果DataGrid屬性ShowCheckBox設為True，如何取到所有被勾選資料的某欄位值?

```javascript
function GetCheck()
 {
   var rows = $("#dgMaster").datagrid('getChecked'); //取出所有被勾選的資料
   for(var i = 0 ;i <rows.length;i++){
     alert(rows[i].出貨編號);
   }
 }
```

---

### [#778] Datagrid如何依照有勾選的資料進行欄位加總值的計算?

```javascript
$(function (){          //     第一次進入畫面     先將     total     歸     0          setTimeout(function (){              $("#dgMaster ").find('tfoot >tr >td ').each(function () {                  if ($(this).attr('data-field ') == "     小計     ") {                      $(this).find('.table-cell-text ').html(0);                  }              });          },100);          //     添加     onclick     事件          $('#dgMaster ').click(function(){              var Total = 0;              var sumTotal = 0;              var rows = $("#dgMaster ").datagrid('getChecked '); //     取出所有被勾選的資料              if (rows.length >0){                  for(var i = 0 ;i <rows.length;i++){                      Total=(parseInt(rows[i].     小計     ));                      sumTotal += Total;                  }              }              $("#dgMaster ").find('tfoot >tr >td ').each(function () {                  if ($(this).attr('data-field ') == "     小計     ") {                      $(this).find('.table-cell-text ').html(sumTotal);                  }              });          });       });
```

---

### [#793] 在DataGrid開放編輯中, 如何用"Enter"鍵, 來自動跳往下一筆, 編輯同一個欄位?

```javascript
在該網頁上, 貼入如下的JS: $(function(){   $('#dgMaster ').on('keydown ', 'input.form-control ', function(){     if(event.which == 13) // 按下Enter的話     {       var field = $(this).closest('td ').data('field ');       if(field == '你的欄位 ') // 指定的欄位       {         var count = $('#dgMaster ').datagrid('getRows ').length;         var editIndex = $('#dgMaster ').datagrid('options ').editIndex;         if(editIndex + 1 <count) { //不是最後一筆, 進入編輯           $('#dgMaster ').datagrid('endEdit ', function(){             $('#dgMaster ').datagrid('beginEdit ', editIndex + 1);             var obj=$('#dgMaster ').find('tbody >tr:eq('+ (editIndex +1) + ')').find('td[data-field="你的欄位 "]').find('input.form-control ');             obj.focus();            });          }       }     }   }) })
```

---

### [#807] datagrid 直接編輯時 combobox 如何取得TextField的值?

```javascript
('#dgDetail ').find('.selected ').find('[data-field="欄位名稱 "]').find('.form-control option:selected ')).text();
```

---

## 欄位顯示控制

### [#585] 如何在DataGrid的欄位中顯示多行(使用 Multiline的格式但顯示不會換行)?

請在datagrid的該欄位中設定Formatter事件，產生如下的js程式即可:

```javascript
 function dgMaster_地址_formatter(value, row, index)
 {
```

value=value.replace(/\n/g,'<br >');

```javascript
   return value;
 }
```

---

### [#588] Image 在datagrid 如何控制照片大小(寬度)?

設定 DataGrid的相片 Format屬性即可，如下:
image,Images,200  (200代表相片寬度)

---

### [#603] 如何讓DataGrid  TextArea的內容換行顯示如同在DataForm顯示的樣子?

請在DataGrid該欄位的Formatter設定一個Function名稱
然後請至 "原始碼" 寫下以下的function

```javascript
     function test(value, row, index)
     {
       return value.replace(/\n/g,"<br />");
     }
```

---

### [#679] 如何讓DataGrid某欄位的數值隨著條件變更顏色且依然使用原先的format?

舉例如下:

```javascript
 function dgMaster_總計含稅_formatter(value, row, index)
 {
   if (value &&value >50000 ){
     return '<span style="color:red ">'+ $.getFormatValue.call(this, value, row, $(this)[0].format) + '</span >';
   }
   else {
     return $.getFormatValue.call(this, value, row, $(this)[0].format);
   }
 }
```

---

### [#682] DataGrid的欄位寬度通常是自動調整的,如何控制某些欄位讓他固定的寬度,以滿足使用者輸入的要求?

```javascript
RWD的DataGrid是自適應寬度, 所以需透過CSS來改變欄位寬度, 可在EEPCloud中貼入Literal網頁組件, 並以CSS的方式在Html屬性中來改變欄寬, 如下:  <style >     #dgMaster th[data-field="客戶名稱 "]{    width: 100px!important;     }  </style > 註: 上面的data-field代表欄位名稱。
```

---

### [#696] DataGrid編輯時, 如果有些欄位為ReadOnly狀態, 可否使用原來顯示的格式, 而非ReadOnly的文字編輯框?

可以透過 DataGrid的 onShowEditor事件來處理, 如下:

```javascript
 function dgDetail_onShowEditor(index, field, editor)
 {
   if(field == '題目'){ //欄位為'題目'時, 改育 type: 'div'來顯示
     return {type: 'div'};
   }
   else  {
     return editor; // 使用原來的 Editor
   }
 }
```

---

### [#703] 如何動態顯示或隱藏DataGrid的欄位?

舉例如下:

```javascript
 //隱藏
 $("#dgMaster").datagrid('hideColumn', '姓名');
 //顯示
 $("#dgMaster").datagrid('showColumn', '姓名');
```

---

### [#705] 如何動態修改DataGrid的欄位Caption?

舉例如下:

```javascript
 //RWD 一般畫面 改變欄位title
   $("#dgMaster").find('thead >tr >th').each(function () {
         if($.parseOptions(this).field == "英文姓名"){
```

this.innerText = "測試1";

```javascript
           //this.style.color = "red";
           //this.style.fontSize = "22px";
         }
       });
   //RWD 手機/平板畫面 改變欄位title
   $("#dgMaster").find('tbody >tr >td').each(function () {
         if ($(this).attr('data-field') == "員工編號") {
           $(this).find('.table-cell-label').html("測試2");
           //$(this).find('.table-cell-label').css('color',"red");
           //$(this).find('.table-cell-label')[0].style.fontSize = "22px";
         }
       });
```

---

### [#723] 請問 DataGrid 的顯示欄位上, 如何放入提示訊息(Title訊息)?

可以使用 Formatter屬性來控制, 如下為使用Title訊息顯示另一個欄位內容:

```javascript
 function dgMaster_請假日期時間_formatter(value, row, index)
 {
   var data = $.getFormatValue.call(this, value, row, $(this)[0].format); //以原來的Format來顯示
   var title = '結束日期時間:'+$.getFormatValue.call(this, row.結束日期時間 , row, $(this)[0].format); // 取得另一個欄位
   return '<span title="'+ title +'">' + data  + '</span >';
 }
```

---

### [#725] 如何控制DataGrid Row的背景顏色?

透過DataGrid的RowStytler事件來處理, 如下:
js舉例如下:

```javascript
 function dgMaster_rowStyler(index, row)
 {
   if(index%2 == 0)
     return 'background-color:#FFFFBB;color:black;';
```

else

```javascript
     return 'background-color:#77FFCC;color:black;';
 }
```

---

### [#727] 如果不希望DataGrid的某欄位自動換行，該如何處理?

設定DataGrid->Columns->該欄位的屬性Nowrap設為false即可

---

### [#799] 如何讓DataGrid某欄位隨著條件變更顏色且依然使用原先的relation?

```javascript
舉例如下: function dgMaster_報價單號_formatter(value, row, index) {   if (row.總計含稅 &&row.總計含稅 >50000 ){      return '<span style="color:red ">'+ $.fn.datagrid.methods.formatDisplay.call(this, value, row); + '</span >';    }    else {       return $.fn.datagrid.methods.formatDisplay.call(this, value, row);   } }
```

---

### [#800] 如果fileupload使用的DataType為blob,datagrid 該欄位該如何設定format?

顯示圖片: image,blob 檔案下載: file,blob

---

## 編輯與存檔

### [#609] 如何在DataGrid中, 只更新一筆數據?

```javascript
$(target).datagrid('updateRow', {
```

index: index,

```javascript
           row: { fieldName: 'value'}//可以多欄位 用','隔開
           });
```

target ->DataGrid的ID
fieldName ->欄位名稱
'value' ->給予的值

---

### [#610] 如何在DataGrid中, 只重新載入當頁數據?

```javascript
var page = $('#dgMaster').datagrid('options').page;
 $('#dgMaster').datagrid('load', { page: page});
```

---

### [#613] JQuery 頁面的DataGrid如何動態的disable和enable欄位?

```javascript
//修改時，欄位不可編輯
 function updateRow(row)
 {
   var datagrid = $(this);
   var row = datagrid.datagrid('getSelected');
   var index = datagrid.datagrid('getRowIndex', row);
   datagrid.datagrid('end_edit', function() {
     datagrid.datagrid('options').editIndex = index;
     datagrid.datagrid('selectRow', index).datagrid('beginEdit', index);
     var cellEdit = $('#dgMaster').datagrid('getEditor', { index: index, field: 'FieldName' });
     $(cellEdit.target).textbox('disable');//TextBox
           $(cellEdit.target).textbox('enable');//TextBox
           //$(cellEdit.target).combobox('disable'); //combobox
           //$(cellEdit.target).datebox({ disabled: true }); //datebox
           //$(cellEdit.target).refval('readonly',true); //refval
   });
   return false;
 }
```

---

### [#615] 如何控制 DataGrid的更改與刪除是否可以執行?

```javascript
function dgMaster_onUpdate(row)
 {
   var date = row.進貨日期.substr(0,4)+row.進貨日期.substr(5,2);
   var closeym = $.getVariableValue('closeYM');
   if (date <= closeym) {
    $.alert("進貨日期小於結帳日, 無法刪除與更改",'info');
    return false;
   }
   return true;
 }
```

---

### [#619] 如何新增時複製DataGrid的某一筆資料?

請在DataGrid的ToolItem添加一個按鈕，
OnClick 呼叫 copy_row

---

### [#620] 如何讓DataGrid與DataForm同時顯示，並兩者互相同步?

只要把DataGrid的EditForm清空，並將DataForm的Mode設為Panel後，再設定 DataGrid的 onSelect事件就可以讓 DataGrid與DataForm同時與同步顯示: 如下:

```javascript
 function dgMaster_onSelect(index, row)
 {
   $('#dfMaster').form('setWhere', "F001=" + row.F001);
 }
```

如果EditForm要能夠編輯，可以設定ToolItem內的Add/Update/Submit/Cancel等Button即可，並設定onApplied事件去同步DataGrid的內容，如下:

```javascript
 function dfMaster_onApplied(data)
 {
   var page = $('#dgMaster').datagrid('options').page
   $('#dgMaster').datagrid('load', {page: page});
 }
```

---

### [#668] 如何在DataGrid的編輯中, 讀取欄位的編輯內容, 及改變某欄位的內容?

可以透過Columns中的個別事件讀取與設定DataGrid欄位內容, 如在Refval或Combo可以透過onSelect事件, Textbox可以透過onBlur事件來處理, 如下範例:

```javascript
 function dgDetail_項目次序_onSelect(row)
 {
   var type = $('#dgDetail').datagrid('getEditorValue', '類別'); //讀取欄位值
   var amt = 0;
   if (type=="代扣所得") {
     amt = -($('#dptab_薪資設定_所得扣繳金額').val());
   }
   else if (type=="代扣保險") {
     amt = -($('#dptab_勞保健保資料_本人勞健保費').val());
   }
   $('#dgDetail').datagrid('setEditorValue',{field: '金額', value: amt}); // 設定欄位值
 }
```

---

### [#729] 如何讓DataGrid結束編輯?

舉例:
在明細DataGrid的Toolitem添加一個按鈕去執行下方的js

```javascript
 function end_Edit()
 {
   $("#dgDetail").datagrid('endEdit');
 }
```

---

### [#753] DataGrid編輯時, 是否可以動態改變編輯器的屬性內容?

可以的, 如下使用 onShowEditor事件來處理, 利用editor來存取編輯器的屬性, 下面的例子是動態改變Combobox的欄位與條件:

```javascript
 function dgDetail2_onShowEditor(index, field, editor)
 {
   if(field == '聯絡人') {
```

editor.options.valueField = '次序';
editor.options.textField = '姓名';

```javascript
     editor.options.whereItems = [{ field: '客戶編號', operator: '=', value: "parent['客戶編號']"}];
   }
   return editor;
 }
```

---

### [#755] 如何自行用程式進行DataGrid的新增/更改/刪除功能?

舉例如下:

```javascript
 var index = $('#dgMaster').datagrid('getSelectedIndex');
 $('#dgMaster').datagrid('viewRow', index); // 查看
 $('#dgMaster').datagrid('delete_row', index); // 刪除
 $('#dgMaster').datagrid('edit_row', index); //更改
 $('#dgMaster').datagrid('insert_row'); //新增
```

瀏覽和編輯方法的第二個參數是rowindex，上面'getSelectdIndex是代表選中的row。 如果要指定某一筆row可以自行設定index值。

---

### [#774] 如果我想在DataGrid的編輯要帶入初值, 而不是新增時帶入, 要如何處理?

```javascript
可以使用     DataGrid     的     onUpdate     事件來處理     ,      以下為上一筆欄位的內容值     ,      帶入下一筆的方法     :       function dgDetail_     摘要     _onBlur()        {          var rem = $('#dgDetail ').datagrid('getEditorValue ', '     摘要     ');          $.setVariableValue('remark ',rem,true); //      設定全域變數     remark       }               function dgDetail_onUpdate(row)       {          if (row.     摘要     .length==0) row.     摘要     = $.getVariableValue('remark ');          return true;       }
```

---

### [#782] 如何在DataGrid編輯某個欄位之後, 觸發事件來處理其他欄位內容?

```javascript
建議直接使用我們的     FieldOnBlur     的設定通常可以直接達到效果，真的非寫不可的話     ,      不同組件     (Editor)     會有不同的離開事件     ,      如下     :       TextBox     有     : onBlur       Refval/Combol/Options/DataBox/DateTimeBox     有     : onSelect       Signature     有     : onChange       Scan     有     : onScan       透過以上的事件     ,      就可以，如下     ,      欄位     1     輸入完畢     ,      自動傳值給欄位     2:       function dgMaster_     欄位     1_onBlur()       {         var editor=$(this).closest('tr ').find('td[data-field="     欄位     2 "]').find('.form-control '); //     取得對方欄位對象         editor.val($(this).val()); //      把欄位     1     的內容設定給欄位     2       }
```

---

### [#783] 如何在DataGrid編輯某個欄位之後, 觸發事件來動態過濾Combobox欄位選項的內容?

```javascript
舉例     :       function dgMaster_     產品類別     _onSelect(row)       {          var editor=$(this).closest('tr ').find('td[data-field="     單位     "]').find('.form-control '); //     取得     單位          editor.combobox('setWhere ',"     類型     !=N '     個     '");//      動態     setWhere       }
```

---

### [#784] DataGrid編輯中, 如何透過Refval選不同的ID, 來控制其他欄位是否可以編輯?

```javascript
可以透過 Refval的 onSelect事件來控制, 如下: function dgDetail_科目編號_onSelect(row) {   if (row.性質控制!='是 ') { // 判斷條件    $(this).closest('tr ').find('td[data-field="性質別 "]').find('.form-control ').setReadonly(true); // 將性質別設為Readonly   }   else {    $(this).closest('tr ').find('td[data-field="性質別 "]').find('.form-control ').setReadonly(false); // 將性質別設為可編輯   } }
```

---

### [#798] 為何會遇到存檔時顯示 無效資料行名稱 "自動編號" ?

此問題可能是因為主明細檔
明細datagrid有設定DuplicateCheck為true
且 主檔Key欄位為 int identity 但前端預設值賦予文字(自動編號)
此時存檔當下,系統會先拿主檔的key值內容 &明細本身的key去檢查是否有重複資料,
但因為主明細關聯的欄位為int
因此在SQL組出的where條件不會替主檔Key值用單引號包起來
所以才會發生這個錯誤
解決方式
1.把主檔Key欄位預設改為 0 (數字)
或
2.把明細的 DuplicateCheck 改為false

---

### [#805] 當使用datagrid編輯時，如何即時知道正在編輯的資料是否通過validate檢核?

```javascript
endEdit              方法除了結束編輯狀態外，還能接收boolean         值，                     根據true/false         判斷當前編輯的資料是否通過            EX:       var result = $('#id ').datagrid('endEdit ');
```

---

## 查詢與過濾

### [#582] 查詢窗體中的Combobox如何額外設定Where的條件?

在DataGrid的QueryColumn中，找到Combobox元件，並在OnBeforeLoad事件裡點兩下，會自動產生以下的方法:

```javascript
     function dgMaster_CustomerID_onBeforeLoad(param)
     {
                       param.whereStr = "Code like 'A%'" //你的where條件
     }
```

---

### [#635] 如何以Tree組件自由控制DataGrid的Where條件?

透過 Tree組件的onNodeSelected事件來實現, 如下:

```javascript
 function Tree1_onNodeSelected(event, node)
 {
   var sID = node.row.項目序號;
   var dataGrid = $('#dgDetail');
   dataGrid.datagrid('setWhere',"(項目序號='"+sID+"' or 上層項目='"+sID+"')");
 }
```

---

### [#636] DataGrid如何預設打開的顯示條件?

透過onBeoforeLoad事件來改變Where條件即可。如下:

```javascript
 function dgMaster_onBeforeLoad(param)
 {
   var user = $.getVariableValue('user');
   param.whereStr = "交辦人 = '" + user + "' or 待辦人='" + user + "'";
 }
```

---

### [#683] 如何用JS程式取得DataGrid的查詢條件?

可以透過Runtime頁面的查詢元件以右鍵的'檢查'來得知查詢物件的ID, 再透過如下的JS來取得查詢內容:

```javascript
 var yymm = $('#dgMasterqueryObj_薪資年月').dateselect('getValue');
 var co = $('#dgMasterqueryObj_子公司').combobox('getValue');
 var all = $('#dgMasterqueryObj_全部印出').selectoptions('getValue');
```

---

### [#690] 如何動態更改DataGrid的資料源(RemoteName), 來達到動態的統計效果?

例如, 原來資料為月份統計的明細, 可以透過一個 '年度彙總' 改為年度的統計內容, 如下:

```javascript
 function reLoad()
 {
   $('#dgMaster').datagrid('options').remoteName = '薪資發放作業.薪資發放統計';
   $('#dgMaster').datagrid('load');
 }
```

---

### [#700] 如何使用 Stored Procedure來查詢自訂的資料, 並傳回到DataGrid中?

* InfoCommand的CommandType如果為StoredProcedure時，前端的DataGrid查詢條件送回後端時會自動與StoredProcedure配合。如果在InfoCommand的Parameters中所設定的參數與前端DataGrid欄位名稱一致時，前端送過來的查詢條件會自動寫入Parameters的Value中，這樣就可以透過StoredProcedure的Input變數進行資料查詢與回傳結果。如下:
1. 設計一個SP，透過MSSQL或是EEPCloud來新增一個SP，並定義好input參數。
2. 原來前端的DataGrid需事先連上一個虛擬或實體的Table，結構與SP要傳會的資料一致。在Server端除了原來的InfoCommand對應到這個虛擬或實體的Table外，還要另外貼一個SP的InfoCommand(CommandType為StoredProcedure)，CommandText輸入SP的名稱，並填好與SP對應的Parameters參數。
3. 前端的DataGrid.RemoteName改成這個SP的InfoCommand即可，這樣User輸入查詢條件就會送往SP這個InfoCommand，InfoCommand因為為SP的Type，所以會自動用欄位對應Parameters參數，因此infoCommand的parameter value不需要再設定為 @xxx

---

### [#750] DataGrid的查詢, 我如何更換查詢條件?

可以透過 onQuery事件, 如下:

```javascript
 function dgMaster_onQuery(whereItems)
 {
```

for (i=0;i <whereItems.length;i++)

```javascript
   {
     if (whereItems[i].field=='帳戶' &&whereItems[i].value=='全部') whereItems[i].value=""; // 如果是全部, 代表查詢為空
   }
   return true;
 }
```

---

### [#751] DataGrid查詢時, 我如何在欄位名稱上加上運算式來查詢?

前端的 onQuery是無法改動欄位名稱或加上運算式的, 所以必須改用後端 InfoCommand的onBeforeExecuteSQL事件來處理, 如下:
exports.股東開會通知書_onBeforeExecuteSQL = function(sql, whereStrs)

```javascript
 {
   for (var i = 0; i <whereStrs.length; i++) {
    whereStrs[i] = whereStrs[i].replace(/股東戶號/g, "CONVERT(INT,股東戶號)"); //將股東戶號改用INT的模式來查詢
   }
   return sql;
 };
```

---

### [#756] 如何自行控制DataGrid的Query視窗所執行的"查詢/清除取消"等按鈕功能?

在DataGrid的onBeforeLoad事件處理，舉例如下:
改寫DataGrid Query "查詢" 按鈕要做的事:

```javascript
 $('.form-query').unbind('click').bind('click', function () {
   //$('#dgMasterqueryObj').form('query'); //原來執行的功能
   alert('Query');
 });
 //改寫DataGrid Query "清除"按鈕要做的事情:
 $('.form-clear').unbind('click').bind('click', function () {
   // $('#dgMasterqueryObj').form('clear'); //原來執行的功能
 alert('Clear');
 });
 //改寫DataGrid Query "取消"按鈕要做的事情:
 $('.form-close').unbind('click').bind('click', function () {
   $('#dgMasterqueryObj').form('close'); //原來執行的功能
   alert('Close');
 });
```

---

### [#763] 如何隱藏或打開Query查詢的Panel?

```javascript
$('#dgMasterqueryObj_panel').collapse('hide'); // 隱藏
 $('#dgMasterqueryObj_panel').collapse('show'); // 打開
```

---

### [#765] DataGrid打開的查詢頁面過小, 會造成部分查詢欄位折行, 可以加大彈出頁面的寬度嗎?

可以, 統一在"網頁"/"共用樣式表(CSS)"中, 設定以下的CSS內容即可:

```javascript
 @media (min-width: 768px){
   .bootstrap-form[id$="queryObj"] .modal-dialog{
     width: 700px; // 電腦桌面開啟寬度改為700點
   }
```

---

### [#773] 如何在 DataGrid的QueryColumns中固定增加一個欄位?

```javascript
有兩個方法     ,      一個寫在前端     DataGrid.onQuery()      事件     ,      如下     :         whereItems.push({field:'     子科目     ',operator: '!=',value: ''});            return true;       }       另一個方法寫在後端的     Command.onBeforeExecuteSQL     事件中     ,      如下     :       exports.     明細分類帳     _onBeforeExecuteSQL = function(sql, whereStrs)       {          whereStrs.push("     子科目     <>''");          return sql;       };
```

---

### [#775] 如何利用ToolItem對datagrid的資料進行過濾?

```javascript
可以在     ToolItem     添加一個按鈕，     onClick     時呼叫一個     function     ，參考如下     :       function test(){                   $('#dgMaster ').datagrid('setWhere ',"     您的     Where     條件     '");            // dgMaster     為     datagrid id               }
```

---

### [#779] 如何在Query畫面，在某一個欄位按下Enter鍵後，執行查詢動作?

```javascript
程式碼參考如下: $(function(){        $('#dgMasterqueryObj     _名稱 ').bind('keypress ',function(event){    If (event.keyCode == ''13 '') { //13代表”Enter”鍵     $('#dgMasterqueryObj ').form('query ');   } }) });
```

---

## UI與樣式

### [#581] 如何使DataGrid的欄位可以排序?

進入EEPCloud畫面，左側點選RWD(Boostrap)打開表單，點選dgMaster，再點選 Columns ，
最後選擇您要排序的欄位，並將 Sortable 打勾即可。

---

### [#584] 如何控制DataGrid上ToolItem隱藏或顯示?

```javascript
如下在表單的 JS中，增加下列程式來隱藏 "新增 "的按鈕 (新增的onclick為 "insert_row ") $('#dgMaster ').datagrid('getToolItem ','insert_row ').hide();
```

上面的
dgMaster  ->datagrid 的ID
insert_row  ->toolitem onClick呼叫的function名稱

```javascript
     $(this).hide(); ->toolitem隱藏
     $(this).show(); ->toolitem 顯示
```

---

### [#664] 如何動態隱藏某一個DataGrid?

如下的程序:

```javascript
 $('#dgMaster ').closest('div ').parent().hide();//隱藏datagrid
```

---

### [#677] 為何DataGrid裡的ToolItems中的OnClick無法讓我鎖定成我更改過後的,每次iCoder產生時都會把我改的OnClick覆蓋?  

ToolItems的OnClick屬性為版本差異比對的主要key值, 所以無法鎖定,每次Word輸出都會被覆蓋, 如果不想被覆蓋的話，請直接添加一個新的按鈕，這樣重新匯入Word時就不會覆蓋掉OnClick。

---

### [#695] 如何用js動態隱藏DataGrid?

舉例如下:

```javascript
 $('#dgMaster').closest('div').parent().hide();//隱藏DataGrid
 //$('#dgMaster').closest('div').parent().show();//顯示DataGrid
```

---

### [#704] 如何動態顯示或隱藏 DataGrid的查看/編輯/刪除的按鈕?

在DataGrid的OnBeforeLoad事件添加如下的js

```javascript
 function dgMaster_onBeforeLoad(param)
 {
 //隱藏
   $('#dgMaster').datagrid('options').viewCommandVisible = false;
   //如果要顯示將false改為true
   //$('#dgMaster').datagrid('options').editCommandVisible = false;
   //$('#dgMaster').datagrid('options').deleteCommandVisible = false;
 }
```

---

### [#754] 如何在控制DataGrid動態隱藏或顯示每一筆row的(查看/編輯/刪除)按鈕?

可以透過OnLoad事件來處理，如下的方式:

```javascript
 $($('#dgMaster').find('.glyphicon-eye-open')[i]).hide();//查看按鈕
 $($('#dgMaster').find('.glyphicon-pencil')[i]).hide();//編輯按鈕
 $($('#dgMaster').find('.glyphicon-remove')[i]).hide();//刪除按鈕
 // 以上i就是row的序號，從0開始，如要顯示的話將hide()改為show()
```

---

### [#757] 如何動態改變DataGrid 的標題Title?

舉例如下:

```javascript
 $('#dgMaster').prev().prev()[0].innerText = "你的標題";
```

---

### [#759] 如何取得DataGrid的資料總筆數?

可以在DataGrid_OnLoad的時機點取得資料總筆數，如下:

```javascript
 function dgMaster_onLoad(data)
 {
     alert(data.total);
 }
```

---

### [#769] 如何搭配ShowCheckBox的屬性控制當頁資料全部勾選或者全部取消勾選?  

舉例:

```javascript
 function CheckAll()
 {
   //全部勾選
   $(this).datagrid('checkAll');
 }
 function UnCheckAll()
 {
   //全部取消勾選
   $(this).datagrid('uncheckAll');
 }
```

---

### [#770] 如何讓user輸入密碼後才允許刪除?

舉例:
請先將DataGrid的ConfirmDelete屬性設為False，再以onDelete的事件處理如下

```javascript
     var delFlag = false;
     function dgMaster_onDelete(row)
     {
       var delPassword='';
       if (!delFlag) {
         $.prompt( "請輸入管理者刪除密碼: ?","" , function(DelPassword) {
           if (DelPassword =='123'){ //為了保密, 這段最好寫在後端
             var index = $('#dgMaster').datagrid('getSelectedIndex');
```

delFlag = true;

```javascript
             $('#dgMaster').datagrid('delete_row', index);
           }
           else {
             $.alert('密碼不正確, 取消刪除','info');
```

delFlag = false;

```javascript
           }
         });
       }
       return delFlag;
     }
```

---

### [#776] 為何我資料庫欄位前面故意有空格, 但到了DataGrid卻不見了?

```javascript
因為空格在網頁中是無法顯示的     ,      要使用     HTML     的格式才會顯示     ,      可以加入     formatter      事件來處理     :       function dgMaster_     科目名稱     _formatter(value, row, index)       {        return (value || '').replace(/\s/g, '&nbsp;');          }
```

---

### [#777] 如何默認選取Datagrid指定的第幾筆資料?

```javascript
function select_first()       {          var index = 0;          $('#dgMaster ').find("tbody >tr:eq("+index+")").addClass('selected ').addClass('info ');          //$('#dgMaster ').find("tbody ").find("tr:eq("+index+")").addClass('selected ').addClass('info ');       }
```

---

### [#780] 如何默認選中datagrid第一筆資料?

```javascript
function select_first()       {          var index = 1; //     第一筆資料          $($('#dgMaster ').find('tr ')[index]).addClass('selected ').addClass('info ');       }
```

---

### [#790] 請問, DataGrid如何自由控制, 前面三個小按鈕(查看/編輯/刪除)的出現樣式?

可以直接設定DataGrid的Simple屬性, 設定為True, 會以浮動的方式顯示這三個小按鈕, False時會直接同時顯示。

---

### [#791] 如何讓drilldown點擊時透過判斷去開啟另一個drilldown元件指定的頁面?

```javascript
可以利用     formatter     事件處理     ,       舉例如下     :         function dgMaster_出貨編號_formatter(value, row, index) {   if(value == 'S2108001 '){  return $.getFormatValue.call(this, value, row, 'drilldown,#Drilldown1 ');    }   else{     return $.getFormatValue.call(this, value, row, 'drilldown,#Drilldown2 ');   } }
```

---

### [#802] 當Datagrid上欄位過多出現滾輪時，往右邊滾動grid的title不會自動補滿，請問如何調整?

```javascript
可以在datagrid onLoad的時機點添加以下程式碼 $('#dgMaster ').parent()[0].style.display = 'grid '; //ID請自行替換
```

---

### [#806] 當datagrid資料過多，或是有設定datagrid高度，導致出現滾輪時，希望可以自動將螢幕滾動至特定資料行位置，並修改其背景顏色，請問該如何達成?

```javascript
於datagrid onLoad         事件，添加以下程式碼(ID         須留意自行替換)            function dgMaster_onLoad(data)       {          var targetElement = $($('#dgMaster ').children()[1]).find('tr ')[6];           //              找尋dgMaster         下第(6+1)         個元素，也就是第七筆資料               targetElement.style.backgroundColor = 'red ';                                 //     將該筆資料背景色調整          targetElement.scrollIntoView({ behavior: "smooth "});                        //     移至該資料行位置       }
```

---

## 匯出匯入

### [#618] 在DataGrid的印表到Word或Excel上, 可否動態更換套印的Word或Excel檔嗎?

可以的, 透過 DataGrid.ToolItems中, 設定onclick事件, 重新定義套印的Word或Excel, 如Excel為:

```javascript
 function exportExcel2()
 {
           $(this).datagrid('exportExcel ', '銷貨統計表依業務 '); // 更換成 "銷貨統計表依業務.xls "
 }
```

Word則為:

```javascript
 function exportWord2()
 {
           $(this).datagrid('exportWord ', '出貨單2 '); // 更換成 "出貨單2.doc "
 }
```

---

### [#656] 如何在 DataGrid中以 Excel上傳來匯入資料,而不是一筆一筆輸入?

可以在DataGrid的ToolItems中, 新增一個"導入"的Button, 設定為 'importExcel' 即可

---

### [#785] 為何我Word套表時, 改用別的doc, 使用了 exportWord方法, 結果出現"REPORTFORMAT undefine"的錯誤?

```javascript
通常     Word     導入的與印表的格式不同可以用以下語法     :        $(this).datagrid('exportWord ','     活動報名名單     ');              但這個     '     活動報名名單     '     是要另外導入到     iCoder     中的     ,      如果沒有透過導入而是透過     "     指定     word     套表格式     "     上傳者，則須改用以下的方式     :        $(this).datagrid('exportWord ', {wordName: '     活動報名名單     .docx '});
```

---

### [#787] 如何在datagrid欄位自訂formater, 讓此欄位也能進行檔案的下載?


```javascript
請參考一下程式碼: function dgMaster_上傳檔案_formatter(value, row, index) {   value = 'test ';   var value2 = '<a href="javascript:void(0)"onclick="downloadFile(\''+ row.上傳檔案 +'\')"style="color:#337ab7 ">'+value+'</a >';   return value2; } function downloadFile(name){   window.open('../file?q='+ encodeURIComponent(name) +'&f=Files '); //根據指定的資料夾名稱調整 &f }
```

---

### [#794] 請問, 上傳Excel如果資料量很大, 可否有進度顯示?

```javascript
可以, 可以透過 $.loading() 來顯示進度訊息, 並透過DataGrid的onImportExcelSucess事件來關閉訊息, 如下:       function myImport() {          $.loading($('#dgMaster ').closest('div '), '導入中...'); // 顯示進度訊息          $(this).datagrid('importExcel '); // 或者呼叫importExcelNotApply       }
            function dgMaster_onImportExcelSuccess()       {          $.alert('上傳成功!','info ');          $.loaded($('#dgMaster ').closest('div ')) //關閉訊息框       }
```

---

### [#795] 為何我日期使用 Format() 會沒有效果?

```javascript
可能是因為日期的格式已經被轉換為字串了, 字串格式如果要使用Format()請改用dateToStr() 的方法, 如下: function DataList1_標題_formatter(value, row, index) {   var ret=dateToStr(row.日期,'yyyy-MM-dd '); //取代 row.日期.Format('yyyy-MM-dd ')   return ret; }
```

---

### [#796] 為何我DataGrid的欄位設定Format為 barcode, 卻無法顯示?

目前系統提供給barcode         的Foramt         有code39/code128/ean13/ean123         四種，因此Datagrid         欄位的Format         屬性除了barcode         外，還需加上barcode         本身的類型Format         ，
EX: barcode; code39

---

### [#801] 如何自定義輸出的Word/Excel檔名?

```javascript
參考如下:            function exportExcel2(){           $('#dgMaster ').datagrid('exportWordl ',{downloadName:'              自訂檔名'});                //$('#dgMaster ').datagrid('exportExcel ',{downloadName:'              自訂檔名'});            }
```

---

### [#804] 請問當datagrid有設定ShowCheckBox為true，要根據使用者勾選的資料，進行exportWord逐筆輸出，應如何設計?

```javascript
參考如下:            async function myexport() {          var rows = $('#dgMaster ').datagrid('getChecked '); //     取得勾選的資料集合          if (rows.length == 0) {              alert('              請先勾選資料');               } else {              $.confirm('              確認印表?', async function (ok) {                       if (ok) {                      for (var i = 0; i <rows.length; i++) {                          //              透過for         迴圈傳遞被勾選的資料Key         值                               var NO = rows[i].              報價單號;                               //              呼叫rep                               await rep(NO);                       }                  }              });          }       }               const rep = (NO) =>{          return new Promise(function(resolve) {              //              準備另一個datagrid         過濾資料，可以透過JS         設為隱藏，只做套表用                   $('#DataGrid1 ').datagrid('setWhere ', "              報價單號= '"+ NO + "'");                          setTimeout(() =>{                  //              根據Key         值過濾後僅會有一筆資料，透過JS         選定該筆來做exportWord                       $('#DataGrid1 ').find('tbody >tr:eq(0)').addClass('selected ').addClass('info ');                  $("#DataGrid1 ").datagrid("exportWord ", {                      name: '              報價單', //         用來套表的Word         檔名                           downloadName: '              報價單_ '+ NO //         自定義輸出的檔名                       });                  resolve();              }, 2000);          });       }
```

---

## DrillDown與連動

### [#608] 如何控制DataGrid瀏覽選擇時, 可以對應到另一個DataGrid顯示相對資料?

在DataGrid的Toolitem添加一個按鈕去執行ServerMethod
前端的JS

```javascript
 function serverMethod(){
   var row = $('#dgMaster ').datagrid('getSelected '); //取得dgMaster上選中的一筆資料
     $.callMethod('sCustomers ', 'testServerMethod ', {CustomerID: row.CustomerID}, function(result){
         var rows = $.parseJSON(result); //將JSon轉回Object類型提供給Grid顯示
     $('#DataGrid1 ').datagrid('loadData ', {rows:rows,total:rows.length});
     })
 }
```

裡面的callMethod方法內參數依序是：ServerModule的名字、ServerMethod的名字、上傳的參數（如果沒有的話可以填null）、Server端返回時的回調方法
後端的js

```javascript
 exports. testServerMethod = function(param, callback){
   var clientInfo = this.clientInfo;
   var CustomerID =param.CustomerID;
   var sql = "select 名稱 from 客戶資料表 where 客戶編號 = '"+ CustomerID +"'"; //SQL條件句
   this.queryRaw(clientInfo, clientInfo.database, sql, {}, callback);
 }
```

---

### [#623] 如果DataGrid上有兩個欄位分別DrillDown到不同表單時, 會兩個都同時打開追蹤, 請問如何避免?

在 DataGrid上的Format除了選擇"drilldown"格式外, 還要設定其對應的DrillDown組件, 如設定為: "drilldown,#Drill進貨退回單" 代表要追蹤的對應組件。

---

### [#666] 如何在Drilldown時自定參數傳給另一個頁面?

在Drilldown的onClick雙擊添加一個事件，如下:

```javascript
 function Drilldown1_onClick(row, whereItems)
 {
   row.parameter1 = "abc"; //自定一個parameter1 參數
           return true;
 }
```

在目標頁面寫下

```javascript
 $(function(){
   var drillRow = $.getEncryptParameter('drillRow', 'drill'); //取出drillRow的對象
   var myPara1=drillRow.parameter1; // 取出parameter1參數
 });
```

---

### [#781] 如何在 dataGrid中, 動態控制另一個Refval的資料來源與WhereItem內容?

```javascript
假設有當某一個     Refval     更改時     ,      觸發了     onSelect     事件     ,      透過這個事件     ,      來控制另一個     Refval     的     RemoteName     與     WhereItem,      如下     :       function dgDetail_     科目編號     _onSelect(row)       {       var editor=$(this).closest('tr ').find('td[data-field="     子科目     "]').find('.form-control '); //     取得子科目的     Refval     對象         if (row.     對象別     =='     子科目     ') { //      動態設定     refval     的資料來源          editor.refval('options ').remoteName='     會計科目     .     會計子科目     ';          editor.refval('options ').whereItems = [{field:'     科目編號     ', operator: '=', value: "row['     科目編號     ']"}]; //      動態設定     WhereItem         }         else if (row.     對象別     =='     客戶     ') { //      動態設定     refval     的資料來源          editor.refval('options ').remoteName='     客戶資料表     .     客戶子科目     ';          editor.refval('options ').whereItems = [];             }         else if (row.     對象別     =='     廠商     ') { //      動態設定     refval     的資料來源          editor.refval('options ').remoteName='     廠商資料表     .     廠商子科目     ';          editor.refval('options ').whereItems = [];         }                }
```

---

### [#788] DataGrid直接編輯時，為何Combobox欄位設定WhereItem後，要重新進入該筆編輯狀態才有效?

```javascript
因為combobox只會載入一次資料，不像refval每次打開都會重新抓取資料，  因此假如B欄位設定whereItems (A欄位 = XX) 可以在A欄位的OnSelect 或 OnBlur事件 對B欄位Combobox的內容重新抓取一次 舉例: function dgMaster_城市_onSelect(value) {  var editor=$(this).closest('tr ').find('td[data-field="地區 "]').find('.form-control ');//地區 為欄位名稱   editor.combobox('load '); }
```

---

### [#789] 如何動態給clientMove批次新增的datagrid額外的條件?

```javascript
function testOpenmove(){  $('#cmDetail ').clientmove('options ').whereItems = [{field:'產品編號 ', operator: '=', value: "constant['1 ']"}];  $('#cmDetail ').clientmove('openMove '); }
```

---

## 特殊操作

### [#605] 呼叫後端Server Method改變了資料後, 前端的DataGrid如何更新?

呼叫完後端Server Mehod完畢後，請添加以下的程式碼

```javascript
 $('#dgMaster').datagrid('load');
```

---

### [#644] 如何在DataGrid中選擇一筆資料, 然後呼叫後端的處理程序?

在DataGrid中加一個Button, 然後寫一個JS的方法來調用後端的程序, 如下:

```javascript
 function createPo() {
   var sIndex = $("#dgMaster").datagrid("getSelectedIndex");
   if (sIndex >= 0) {
     var pno = $("#dgMaster").datagrid("getRows")[sIndex].訂單號碼; //取出訂單號碼
     $.callMethod('客戶訂單','doCreatePo',{type:1,no:pno},function(result){
       $.alert(result,'info');
     });
    }
   else {
    $.alert('請選擇一筆訂單','info');
        }
 }
```

---

### [#772] 請問Server Method會處理比較久, 如何頁面上告知'處理中'的訊息?

```javascript
可以透過 $.loading() 來處理, 如下:  $.loading($('#dgMaster ').closest('div '), '處理中...'); // 顯示進度訊息       $.callMethod('員工資料表 ','doCloseSalary ',{yymm:row.年月,co:row.子公司,id:row.員工編號},function(result){ // 執行後端檢查方法         $.loaded($('#dgMaster ').closest('div ')) //關閉訊息框         $.alert('員工:'+row.員工編號+'薪資結算成功!','info ');         $('#dgMaster ').datagrid('load ');       });
```

---

### [#786] DataGrid可否不要一開始就Load資料, 即使設定Where 1=0也不要Load?

```javascript
可以利用onBeforeLoad事件的return false來處理, 如下: function DataGrid1_onBeforeLoad(param) {   $.loaded($(this).closest('div ')); // 取消 '載入中 '的訊息   return false; }
```

---

### [#792] 如何透過code 來達成 sortable 的效果(資料排序)?

可以在datagrid onBeforeLoad     事件處理
如下

```javascript
  function dgMaster_onBeforeLoad(param) {      	     param.sort = "產品名稱 ";//欄位名稱      param.order = "desc ";//asc 順排 ,desc 逆排 }
```

---

### [#797] Datagrid ShowCheckbox設為True時,如何根據某欄位內容動態控制Checkbox是否Disabled(Readonly)?

舉例如下:

```javascript
  function dgMaster_rowStyler(index, row) {   if(row.身分證號 == 'A123456789 ')    {     setTimeout (function (){       $('#dgMaster ').find('tbody >tr >td ').find('[type="checkbox "]')[index*2].setAttribute("disabled ", "disabled ");       $('#dgMaster ').find('tbody >tr >td ').find('[type="checkbox "]')[index*2+1].setAttribute("disabled ", "disabled ");     },100);   } }
```

---

### [#803] 當DataGrid開啟showCheckBox功能，如何取得被勾選資料的index?

```javascript
參考如下: var rows = $('#dgDetail ').datagrid('getChecked '); var indexes = []; $("#dgDetail ").find('tbody >tr >td.datagrid-checkbox >:checked ').each(function() {   indexes .push($(this).closest('tr ').index()); });
```

---

## 其他

### [#675] Master/Detail中如何清除所有Detail的DataGrid資料?

```javascript
function delAll() {
   //取消確認框的方法,但如果ConfirmDelete設為False就不用這段
 $.confirm = function (text, callback) {
```

callback(true);

```javascript
 };
 var rows = $("#dgDetail").datagrid('getRows');
 var cnt = rows.length;
 for (var i = cnt - 1 ;i >= 0 ; i--) {
  $('#dgDetail').datagrid('delete_row', 0);
 }
 $('#dfMaster').form('submit'); //如果要自動存檔的話
 }
```

---

### [#726] 如何讓查詢的Refval欄位做到ColumnMatch的功能?

在QueryColumns的refval欄位屬性Editor裡設定OnSelect事件(雙擊可以直接產生一個function)
js舉例如下:

```javascript
 function dgMaster_客戶編號_onSelect(row)
 {
 $('#dgMasterqueryObj_名稱').val(row.名稱);
 //#dgMasterqueryObj_名稱 為另一個查詢欄位的ID
 //row.名稱 為 refval該筆資料的其他欄位值
 }
```

---

### [#771] 當日期欄位類型為varchar8,如何取得當日前七天日期?

```javascript
function time(){   var today = $.getVariableValue('today '); //取得系統今日日期   var addDate = addDays(today,-20);    //取得前二十天   var date = addDate.substr(0,10);     //取出年月日   date = new Date(date).Format('yyyyMMdd ');  //轉為varchar8   return date; }
```

---
