# DataForm 常見問題與技巧（FAQ）

> 來源：infolight.com 官方常見問答（EEP.NET Core）
> 整理日期：2026-04-16
> 共 64 題，依主題分類整理

---

## 欄位取值與設值

### [#634] DataForm中, 如何用JS存取TextBox的內容?

textbox取值的方法為:

```javascript
var year = $('#dfMaster_請假年月').val();


```

textbox設值的方法為:

```javascript
$('#dfMaster_請假年月').val(yymm);
```

---

### [#650] 在DataForm中, 如何取得 #O Options的選擇內容?


```javascript
如下, 可以透過以 selectoptions()的方法來取得:
function dfMaster_onApply()
{
var date1 = $('#dfMaster_申請日期').datebox('getValue');
var date2 = $('#dfMaster_請假日期').datebox('getValue');
var type = $('#dfMaster_請假類型').selectoptions('getValue');
if (type=='特休假' &&date1 <date2) {
$.alert("特休假 至少需前1日申請之",'info');
return false;
}
return true;
}
```

---

### [#663] 如何取得DataForm欄位Refval的TextValue的值?

Refval的取值方式為:

```javascript
$(target).closest('.row ').find('.refval-text ')[0].outerText;


```

例如:

```javascript
$('#dfMaster_客戶編號 ').closest('.row ').find('.refval-text ')[0].outerText;
```

---

### [#667] DataForm中的各種欄位讀取值與設定值方法都不一樣，如下:

TextBox

```javascript
取值:$(target).val();
給值:$(target).val(value);
```

TextArea

```javascript
取值:$(target).val();
給值:$(target).val(value);
```

Combobox

```javascript
取值:$(target).combobox('getValue ');
給值:$(target).combobox('setValue ',value);
```

Refval

```javascript
取值:$(target).refval('getValue ');
給值:$(target).refval('setValue ',value);
```

Options

```javascript
取值:$(target).selectoptions('getValue ');
給值:$(target).selectoptions('setValue ',value);
```

Numberbox

```javascript
取值:$(target).numberbox ('getValue ');
給值:$(target).numberbox('setValue ',value);
```

Datebox

```javascript
取值:$(target).datebox('getValue ') ;
給值:$(target).datebox('setValue ',value);
```

Datetimebox

```javascript
取值:$(target).datebox('getValue ');
給值:$(target).datebox('setValue ',value);
```

Timebox

```javascript
取值:$(target).timebox('getValue ');
給值:$(target).timebox('setValue ',value);
```

Switch

```javascript
取值:$(target).switch('getValue ');
給值:$(target).switch('setValue ',value);
```

Slider

```javascript
取值:$(target).slider('getValue ');
給值:$(target).slider('setValue ',value);
```

DateSelect

```javascript
取值:$(target).dateselect('getValue ');
給值:$(target).dateselect('setValue ',value);
```

FileUpload

```javascript
取值:$(target).fileupload('getValue ');
給值:$(target). fileupload('setValue ',value);
```

---

### [#793] combobox 如何取得TextField的值?

舉例如下:

```javascript
$('#dfMaster_倉庫 option:selected ').text();

```

dfMaster_倉庫 ->ID

---

### [#797] 如何取得 selectoptions 欄位的 Text?

舉例如下:


```javascript
var txt = $('#dfMaster_報價單號 ').next().find('.btn-default.active ').text();
alert(txt);
```

---

## 欄位顯示控制

### [#580] 如何在user編輯時讓某一個欄位無法編輯?

首先，請至EEPCloud中點選左側RWD選到你要實作的那個表單，
並點選要做設定的該欄位的dataform或datagrid，然後點選右側的Columns。




接下來請找到你要設定的欄位，並點選Editor進去做設定。






最後勾選"Readonly"即可，這樣即可以達到使該欄位無法被新增修改。

---

### [#604] 如何讓RWD DataForm的欄位隱藏?


```javascript
//適用欄位類型:textbox/textarea/numberbox/datebox/dateselect/options/combobox/refval
//$('#dfMaster_欄位').parent().parent().hide();
//$('[for="dfMaster_欄位"]').hide();

//適用欄位類型:multiinput
//$('#dfMaster_欄位').parent().parent().parent().hide();
//$('[for="dfMaster_欄位"]').hide();

//適用欄位類型:switch
//$('#dfMaster_欄位').parent().parent().hide();
//$('#dfMaster_欄位').parent().parent().prev().children().hide();

//適用欄位類型:fileupload
//$('#dfMaster_欄位').parent().hide();
//$('[for="dfMaster_欄位"]').hide();
```

---

### [#611] 如何更改DataForm欄位的底色?


```javascript
$(function(){
$('#dfMaster_CustomerID').css('background-color', '#fdd');
})
```

dfMaster_CustomerID ->請自行替換DataFormID + 欄位ID
#fdd ->請自行替換顏色

---

### [#652] DataForm輸入時, 如何控制不同的欄位條件, 要自動控制那些欄位可以輸入(控制Readonly)?

如下例, 用戶選擇 Options條件後, 可以控制另一個欄位是否可以輸入, 設定其Readonly屬性:

```javascript
function dfMaster_請假類型_onSelect(value)
{
if (value=='特休假') {
$('#dfMaster_特休假年度').setReadonly(false);
}
else {
$('#dfMaster_特休假年度').setReadonly(true)
}
}
```

---

### [#653] DataForm有多個頁簽, 如何控制不同的人可以編輯不同的頁簽? (頁簽如何隱藏)

可以透過DataForm的onLoad事件來控制, 並判斷沒有權限的人, 將頁簽隱藏即可, 如下JS程式:

```javascript
function dfMaster_onLoad(row)
{
var groups = $.getVariableValue('groups');
if (groups.indexOf("80")<0) { // 找不到"80"的群組角色
var tabID = 'tabMaster'; // 設定要隱藏的 Tab
var index = 1; // 第幾頁要隱藏 (從0開始算)
$('[href="#'+ tabID + '_'+ index +'"]').hide();
}
}
```

---

### [#661] 如何針對#G Group群組裡的所有欄位進行ReadOnly處理?


```javascript
var fs = $('#dpgroup_借用資訊_M_CU').closest('fieldset');//取得欄位群組
fs.find('.form-control').setReadonly(true);
//設所有欄位為readonly，反之設成false即可
```

---

### [#733] 如何控制 DataForm的欄位標題與內容的寬度?


```javascript
原來DatForm在sm模式下(電腦或平板頁面), 標題佔2/12, 欄位佔 4/12, 所以可以左右可以放2個欄位，如果想把標題加大到 3/12, 欄位縮小到 3/12(兩個一樣寬), 如下的程式:
```

* 更換DataForm的Title與欄位寬度

```javascript
function dgMaster_onLoad(data)
{
$('#dfMaster').find(".col-sm-4").removeClass("col-sm-4").addClass("col-sm-3"); //把所有的 col-sm-4 換成 col-sm-3
$('#dfMaster').find(".col-sm-2").removeClass("col-sm-2").addClass("col-sm-3"); //把所有的 col-sm-2 換成 col-sm-3
}
```

---

### [#768] 如何針對整個DataForm或指定頁籤底下的所有欄位ReadOnly?

如下的程式:

```javascript
//如何Tab頁籤底下的所有欄位設為ReadOnly
$("#tabMaster_1").find('.form-control').setReadonly(true);
//如何將DataForm底下的所有欄位設為ReadOnly
$("#dfMaster").find('.form-control').setReadonly(true);
```

---

### [#774] DataForm在Mode為Panel時, 顯示的狀態為唯讀的狀態, 如何讓他進入編輯狀態？

可以在需要編輯的時候 , 執行下列 JS 語法即可 :

```javascript
$('#dfMaster ').form('status ', 'updated ');
```

或者 :

```javascript
$('#dfMaster ').form('edit_row ');
```

---

## 存檔與驗證

### [#587] 如何以程式控制DataForm的關閉或存檔?


```javascript
$('#dfMaster').form('close'); //關閉dataform
$('#dfMaster').form('submit'); //存檔dataform
```

---

### [#614] DataForm中如何控制在存檔前再次檢查條件, 並不讓user保存? 


```javascript
function dfMaster_onApply()
{


var date = $('#dfMaster_進貨日期').datebox('getValue');
date = date.substr(0,4)+date.substr(5,2);

var closeym = $.getVariableValue('closeYM');
if (date <= closeym) {
$.alert("進貨日期小於結帳日, 無法保存",'info');
return false;
}
return true;
}
```

---

### [#621] 如何取得DataForm的編輯狀態?


```javascript
var status = $('#dfMaster').form('status');
```

status的值如果是 view 代表是編輯狀態為"查看"，
inserted 代表是編輯狀態為"新增"，
updated 代表是編輯狀態為"修改"。

---

### [#624] 只有DataForm(表單)的情況下，可否一打開表單就進入新增狀態，存擋後並顯示該筆新增的資料。

加入DataForm的onLoad事件:

```javascript
var firstTime = true;
function dfMaster_onLoad(row)
{
if(firstTime) {
firstTime = false;
$(this).form('insert_row'); // 進入新增
}
}

```

並在onApplied事件中, 顯示該新增的資料。

```javascript
function dfMaster_onApplied(data)
{
var key = data[0].inserted[0].F001;
$(this).form('setWhere',"F001='"+key+"'");
$.alert('新增成功!! 編號:'+key,'info');
}
```

---

### [#702] DataForm輸入後, 如何取得自動編號或存檔之後的值?


```javascript
可以使用 onApplied() 事件來處理, 如下:
function dfMaster_onApplied(data)
{
if (data[0].inserted.length >0) // 新增才處理
{
var row=data[0].inserted[0]; // 取得剛剛insert的那一筆資料
var repno = row.報修編號;  // 取得報修編號
var date = row.報修日期;
var man = row.報修人;
$.callMethod('報修服務紀錄','sendMail',{repno1:repno,date1:date,man1:man},function(err,result)
{
if(err)
{
alert(err);
}
else $.alert('已報修並發送給管理中心','info');
}
});
}
return true;
}
```

---

### [#740] 可以控制 DataForm存檔後不關閉嗎? 為了能及時存檔並取得最新的資料。

可以，如下的JS程式:

```javascript
$(function(){
$('#dfMaster').on('hide.bs.modal', function(e) { // DataForm關閉時執行
if($(this).data('isApply')){ // 判斷 isApply是否為True
e.preventDefault(); //取消關閉DataForm的動作
$(this).removeData('isApply'); // 再把isApply變數清除
}
});
})
function dfMaster_onApply()
{
$(this).data('isApply', true); //存檔前設定 isApply變數為True
return true;
}
```

---

### [#778] Validate元件如果透過function檢查資料，如何動態修改錯誤訊息?

舉例 :

```javascript
function v_test(value){
$(this)[0].validator.arguments[1][1] = ' 錯誤訊息 ';
return false;
}
```

---

### [#794] 如何透過code觸發validate元件檢查?

舉例如下:


```javascript
$('#dfMaster ').form('validate ');
```

---

### [#798] 如何動態控制validate元件為function時的Message提示訊息?

舉例如下 :

```javascript
function validateColumn(val){
If(val != '123 '){
this.message = ' 檢核不通過提示訊息 ';
return false;
}else{
return true;
}
}
```

---

## 動態條件控制

### [#746] 如何透過JS動態控制DataGrid的Where條件?

舉例如下:

```javascript
function dgMaster_onBeforeLoad(param)
{
Param.whereStr = “ 你的Where 條件”;
}
```

---

### [#747] 如何透過JS動態控制Refval的Where條件?

舉例如下:

```javascript
function dfMaster_onLoad()
{
$('#dfMaster_欄位').refval('setWhere',"你的where條件");
}
```

---

### [#748] 如何透過JS動態控制Combobox的Where條件?

舉例如下:

```javascript
function dfMaster_onLoad()
{
//Combobox選項來源必須來自RemoteName而非Item
$('#dfMaster_欄位').combobox('setWhere',"你的where條件");
}
```

---

### [#783] DataForm如何對Combobox欄位重新載入資料? 


```javascript
$(target).combobox('load ');

```

舉例:

```javascript
$('#dfMaster_地區 ').combobox('load ');
```

---

### [#784] DataForm如何對Combobox欄位重新載入資料? 

因為combobox只會載入一次資料，不像refval每次打開都會重新抓取資料，


```javascript
因此假如B欄位設定whereItems (A欄位 = XX)
```

可以在A欄位的OnSelect 或 OnBlur事件 對B欄位Combobox的內容重新抓取一次
舉例:

```javascript
function dgMaster_城市_onSelect(value)
{
var editor=$(this).closest('tr ').find('td[data-field="地區 "]').find('.form-control ');//地區 為欄位名稱
editor.combobox('load ');
}
```

---

### [#795] 如何動態改變selectoptions欄位的remoteName?

舉例如下:



```javascript
function dfMaster_onLoad(row)
{
$('#dfMaster_報價單號 ').selectoptions('options ').remoteName = '出貨單.ref_報價單2 ';
$('#dfMaster_報價單號 ').selectoptions('load ');

}
```

---

### [#796] 如何對 selectoptions 欄位動態過濾資料來源(setWhere)?

舉例如下:



```javascript
function dfMaster_OnLoad()
{
$('#dfMaster_產品編號 ').selectoptions('setWhere ',"產品類型 = 'A1 '");
}
```

---

### [#801] 請問 Options的Items如何動態給內容?

如下的方式:

```javascript
var items = ' 國小, 國中, 高中, 大學, 研究所';
var rows = items.split(',').map(function(v){
return {value: v, text: v}
});
$('#dfMaster_ 選項內容').selectoptions('loadData ', rows);
```

---

### [#802] 我如何動態設定ComboBox的Items的內容?

如果在DataForm 中, 你可以透過loadData 來動態設定, 如下:

```javascript
$('#dfMaster_ 項目').combobox('loadData ', [{ value: '1 ', text: ' 項目1 '},{ value: '2 ', text: ' 項目2 '}]);
```

如果是在DataGrid 中編輯, 則可以透過onShowEditor 事件來處理, 如下:

```javascript
function dgMaster_onShowEditor(index, field, editor)
{
if(field == ' 項目')
{
return {
type: 'combobox ', options: { items:[{ value: '1 ', text: ' 項目1 '},{ value: '2 ', text: ' 項目2 '}] } };
}
```

else

```javascript
{
return editor;
}
}
```

---

## UI與樣式

### [#633] 如何讓 DataForm的 Pagination可以隱藏不顯示, 只顯示 ToolItems即可?

貼入Literal 組件, 然後設定 HTML內容如下即可:



```javascript
<style >
#dfMaster .pagination{
```

display:none

```javascript
}
</style >
```

---

### [#671] 如何使DataForm所有欄位caption位置都靠左?

先於工具箱拉出一個Literal，並於HTML中寫下CSS:

```javascript
<style >
@media (min-width: 768px) {
.form-horizontal .control-label {
text-align: left;
}
}
</style >
```

---

### [#672] 如何調整某一個DataForm欄位caption位置靠左?

舉例如下:

```javascript
$('[for="dfMaster_員工編號"]').css('text-align', 'left');
```

---

### [#673] DataForm如何動態更換標題及欄位顏色?

舉例如下:
DataForm標題顏色

```javascript
$('[id="modalLabel"]').css('color', 'red');


```

欄位標題

```javascript
$('[for="dfMaster_員工編號"]').css('color', 'blue');


```

欄位值

```javascript
$("#dfMaster_員工編號").css('color', 'green');
```

---

### [#722] 請問如何更換 DataForm 的Title標題?

可以利用 onLoad事件即可, 如下:

```javascript
function dfMaster_onLoad(row)
{
$('#dfMaster').find('.modal-title').html(row.F001); // F001為欄位名稱
}
```

---

### [#735] DataForm使用到了ToolItems時會順便把導航條(操作上下筆)給顯示出來, 可否將導航條給隱藏?

可以, 請貼入一個 Literal組件, 在Html中設定以下的CSS即可:

```javascript
<style >
#dfMaster .pagination{
```

display:none

```javascript
}
</style >
```

---

### [#776] 如何更改 DataForm中的, '確定' 或 '取消' 按鈕的內容?

可以在 DataForm 的 onLoad 事件中 , 控制 :

```javascript
$('#dfMaster ').find('[data-options="onclick:\'submit\'"]').find('.btn-text ').html(' 報名 ');
$('#dfMaster ').find('[data-options="onclick:\'reload\'"]').find('.btn-text ').html(' 取消報名 ');
```

---

### [#789] 如何更換 DataForm的Button的Text內容或控制顯示與隱藏?

如下的程式 :

```javascript
$('#dfMaster ').find('[data-options="onclick:\'submit\'"]').find('.btn-text ').html(' 更改報名 '); // 更改 text 內容
$('#dfMaster ').find('[data-options="onclick:\'delete_row\'"]').show(); // 將刪除的 Button 顯示出來 , 隱藏請使用 .hide()
```

---

### [#791] 當畫面上欄位多的時候, 容易造成User要移動ScrollBar，請問如何調整DataForm欄位間的上下間隙?

可以拉進一個Literal元件，或是透過共用CSS添加以下代碼


```javascript
<style >

.bootstrap-form .form-editor{
margin-bottom: 5px !important;
}
</style >
```

---

### [#800] 請問如何讓DataForm的同一個位置放入兩個欄位，如手機頁面每列只能放一個欄位, 如何放入兩個欄位?

DataForm 的Columns 只要設定其中一個欄位即可, 然後在DataForm 的onLoad 事件中寫:

```javascript
var init = false;
function dfMaster_onLoad(row)
{
var sourceName = "FirstName "; // 要在這個欄位的右方插入新欄位
var newName = "LastName "; // 插入的欄位名稱
if(!init) // 第一次進來
{
var sourceInput = $(this).find('[name="'+ sourceName  +'"]'); // 找到原欄位的對象
sourceInput.closest('.form-editor ').attr('class ', 'form-editor col-xs-6 col-sm-4 '); // 改變排版方式
var newHtml = '<div class="form-editor col-xs-6 col-sm-6 ">'; // 排在右方
newHtml += '<input id="dfMaster_ '+ newName + '"name="'+ newName +'"type="text "data-options="field:\''+ newName +'\'">';
newHtml += '</div >'; // 設定插入欄位的名稱與輸入組件及屬性
$(newHtml).insertAfter( sourceInput.closest('.form-editor ')); // 插入html
init = true; // 避免重複執行
}
if(row[newName]) $(this).find('[name="'+ newName  +'"]').val(row[newName]); // 插入欄位帶入資料內容
}
```

---

## 開窗與導航

### [#583] 如何在DataForm中，新增一個連結或按鈕 打開另表單?

如下的JS範例，在"請假年度"的後方插入一個"請假狀況"的連結，並將申請人與請假年度傳送到"員工休假狀況表"的表當中。


```javascript
$(function(){
$('<a style="vertical-align:middle;display:table-cell;padding-left:10px;cursor:pointer">請假狀況 </ a >').insertAfter($('#dfMaster__請假年度')).click(function(){
if(window.top.addTab){
var user = $('#dfMaster__申請人').refval('getValue');
var year = $('#dfMaster__請假年度').dateselect('getValue');
window.top.addTab('員工休假狀況表','員工休假狀況表', 'bootstrap/員工休假狀況表?year=' + year + '&user=' + user); //打開表單
}
else{
alert('預覽無法打開此網頁')
}
});
})





另一個表單(如員工休假狀況表)的前端js


function dgMaster_onBeforeLoad(param)
{
var year = $.getParameter('year');
var user = $.getParameter('user');
if(year &&user){
param.whereStr = "年度 = '" + year + "' AND 員工編號='" + user + "'";
// 設定 Datagrid的Where條件
}
}
```

---

### [#760] DataForm新增資料後如何自動匯出Word?

透過DataForm的OnApplied事件來處理，程式碼如下：


```javascript
function dfMaster_onApplied(data)
{
if (data.inserted.length >0) { // 當Insert時才處理
$('#dgMaster').children("tbody").children("tr").removeAttr("class"); //離開選中哪一筆
$('#dgMaster').children("tbody").children("tr:last").attr("class", "selected info"); // 選中目前頁最後一筆
$('#dgMaster').datagrid('exportWord'); //調用輸出Word的功能
}
}
```

---

### [#761] 如何控制使用者的游標進入DataForm的某個欄位？

舉例如下:

```javascript
$('#dfMaster_英文姓名').focus(); // 進入英文姓名編輯
```

---

### [#772] 如何透過js關閉DataForm?

舉例:

```javascript
$('#dfMaster').modal('hide');
```

---

### [#775] 在選單設定自訂義的參數後，打開表單會自動開啟第一筆資料，該如何處理?

將DataForm 屬性 isShowFlowIcon 設為 false即可

---

### [#782] 自己新增的表單中貼的DataForm, 為何無法顯示與編輯? 

DataForm 的 Mode 為 Dialog ，需要配合 DataGrid 才能運作 , 如果單獨使用 , Mode 須設定為 Panel 。

---

### [#785] 如何透過js觸發refval的按鈕?


舉例:

```javascript
$('#dfMaster_客戶編號 ').next().children().click();
```

---

### [#787] 請問如何在DataForm的表頭表尾增加自己標籤或Button內容?

可以透過 DataForm 的 onLoad 事件來處理 , 如下 :

```javascript
function dfMaster_onLoad(row)
{
if ($('#test ').length == 0){ // 判斷是否已存在 , 避免重複
$('<p id="test "style="display:inline-block ">&nbsp123 </p >').appendTo($('#dfMaster ').find('#modalLabel ')); // 插入 "123 " 的標題
}
if ($('#test2 ').length == 0){ // 判斷是否存在 , 避免重複
$('<p id="test2 "style="text-align:left ">456 </p >').prependTo($('#dfMaster ').find('.modal-footer ')); // 插入 456 的標題
}
}
```

---

### [#788] 如何在DataForm自訂一個開窗選資料, 然後傳回到原來的表單?

如下三個步驟 :

```javascript
1. 原表單 (DataForm), 針對某個欄位插入一個 Link, 如下 JS:
$(function(){
$('<a style="vertical-align:middle;display:table-cell;padding-left:10px;cursor:pointer "> 打開連結 </ a >').insertBefore($('#dfMaster_ 你的欄位 ')).click(function()
{  $.openForm(' 開窗選擇 ', '../bootstrap/ 你的對話表單 '); // 打開另一個對話表單
});
});
```

2. 原表單增加以下 JS ，用來傳回內容並關閉對方視窗 :

```javascript
function setPhoto(value){
$('#dfMaster_ 照片 ').fileupload('setValue ',value); // 所要傳回的欄位
$('#modal.modal ').modal('hide '); // 關閉 ShowModal 視窗
}

3. " 你的對話表單 " 中增加以下 JS: ( 用 DataGrid.ToolItems 調用 )
function OK()
{
var rows = $('#dgMaster ').datagrid('getRows ');
var index=$('#dgMaster ').datagrid('getSelectedIndex ');
if (index >=0) {
var row=rows[index];
window.parent.setPhoto(rows[index]. 要傳回的欄位 ); // 用上一層的方法傳值和關閉視窗
}
}
```

---

### [#792] 如何在dataform的確定按鈕前，添加一個按鈕執行自定義的程式?

舉例如下:

```javascript
$(function(){
$('<button type="button "class="btn btn-default flow-submit btn-primary "style="">test </button >').insertBefore($('[class="btn btn-primary form-submit "]')).click(function(){
//執行自訂義的程式碼
});
})
```

---

## 特殊元件操作

### [#645] 如何在前端的DataForm編輯時顯示時鐘?

可以透過以下的 JS 程式來完成:

```javascript
var firstTime = true; //僅Load網頁時第一次才執行.
var NowDateTime = ''; //設定日期時間的共用變數
function dfMaster_onLoad(row) // 讓DataForm自動進入新增狀態
{
if(firstTime) {
firstTime = false;
$(this).form('insert_row '); //新增一筆資料
NowDateTime   = $.getVariableValue('now '); //取得本地日期時間
showtime();
}
}
function showtime()
{
var currtime = new Date(NowDateTime ).Format('yyyy-MM-dd hh:mm:ss '); //取得日期時間變數
$('#dfMaster_日期時間 ').val(currtime); // 設定打卡的時間日期
if ($('#dfMaster ').form('status ')=='inserted ') {
var systemTime = new Date(NowDateTime ).getTime()+1000; //將NowDateTime   轉成數值+1000毫秒
NowDateTime   = new Date(systemTime); //重新存回NowDateTime
setTimeout('showtime()',1000); //1秒以後重新執行showtime()
}
}
```

---

### [#693] DataForm中的DefaulValue通常是新增的時後帶入預設值, 如果想要更改的時後也帶入值, 要如何處理?

可以用 DataForm的onLoad事件來處理, 如下:

```javascript
function dfMaster_onLoad(row)
{
var memo = $('#dpMaster2_常規準備事項').val();
if (memo.length == 0) {
var def = $('#dfMaster').form('getDefaultValues').常規準備事項;
$('#dpMaster2_常規準備事項').val(def);
}
}
```

---

### [#771] Signature簽名組件簽名時，可否自動設定簽名日期時間?

可以利用 Sinature的onChange事件來處理, 如下:

```javascript
function dfMaster_簽收_onChange()
{
$('#dfMaster_簽收日期時間').val(new Date().Format('yyyy-MM-dd hh:mm:ss'));
}
```

---

### [#773] 請問, 如何動態控制 Slider的最大與最小值?

如下的案例 :

```javascript
function dfMaster_ 抽獎編號 _onSelect(row)
{
var max = row. 號碼球數 ;
$('#dfMaster_ 號碼 1 ').bootstrapSlider({min: 1,max:max, step: 1, tooltip: 'hide '});
}
```

---

### [#777] 為何我在 DataForm中使用 Chart組件, 為何無法顯示出來?

因為所有的 DataForm 中的組件預設不顯示 , 如果要讓 Chart 顯示 , 須透過 DataForm 的 onLoad 事件來 resize 一下即可 :

```javascript
function dfMaster_onLoad(row)
{
setTimeout(function(){ // 延遲 0.2 秒再執行
var no = $('#dfMaster_ 活動編號 ').val();
$('#PieChart1 ').piechart('setWhere '," 活動編號 ='"+no+"'");
$('#PieChart1 ').piechart('resize ');
},200);
}
```

---

### [#779] 如何動態控制 #O (Options) 停在哪一個選項?

如下:



```javascript
function setType()
{
var hour = new Date().getHours();
if (hour <10) {
$('#dfMaster_出勤類別 ').selectoptions('setValue ','上班 ');
}
else if (hour >9 &&hour <17) {
$('#dfMaster_出勤類別 ').selectoptions('setValue ','公出 ');
}
else if (hour >16 &&hour <21) {
$('#dfMaster_出勤類別 ').selectoptions('setValue ','下班 ');
}
}
```

---

### [#780] 如果只是要到後端取關聯的名稱, 一定要寫Server Method嗎?


```javascript
如果只是單純的取某個資料表的每一個欄位值, 可以透過 getDisplayText() 系統方法來取值, 省去後端的Server Method，如下:
var userMail = getDisplayText('購物車.User ','USERID ','EMAIL ', $('#dfMaster_客戶編號 ').val() ); // 用客戶編號去取USERS的EMAIL欄位內容
$('#dfMaster_電子郵件 ').val( userMail ); // 設值到電子郵件欄位
```

以上需注意: "購物車.User "是一個RemoteName,需事先在 "購物車 "Server端模組中, 設定好 "Select * From Users "這個Command

---

### [#781] 為何我設定Panel的Collapsed無效果? 又如何用 JS 控制 Panel的收合?

Panel 的 Collasped 必須配合 Title 的設定 , 否則無法有收合功能。如果想要用 JS 控制 Panel 的收合 , 如下 :

```javascript
$('#Panel1 ').find('.collapse ').collapse('show ') // 展開
$('#Panel1 ').find('.collapse ').collapse('hide ') // 收合
```

---

### [#786] Text Area欄位如何使用提示文字功能?

可以在dataform OnLoad事件裡添加以下的js


```javascript
//"dfMaster_地址 "自行替換ID
document.getElementById("dfMaster_地址 ").setAttribute("placeholder ","test ");
```

---

### [#790] 如何透過code動態改變dataform datebox 的 format?

舉例如下


```javascript
$(function(){
$('#dpMaster_OrderDate ').datebox('options ').format = 'yyyy/mm/dd ';
$('#dpMaster_OrderDate ').data('datetimepicker ').setFormat( 'yyyy/mm/dd ');
})
```

---

### [#799] 當fileupload DataType為blob時，如何在上傳檔案時動態修改檔名?

舉例如下:

```javascript
function dfMaster_相片_onSuccess(name)
{
let   new_filename =  'test.jpg ';
let new = new_filename  +  ','+ name.split(',')[1];
setTimeout(function() {
$('#dfMaster_相片 ').fileupload('setValue ', new   );
},10);
}
```

---
