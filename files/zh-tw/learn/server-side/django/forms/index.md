---
title: 'Django Tutorial Part 9: Working with forms'
slug: Learn/Server-side/Django/Forms
translation_of: Learn/Server-side/Django/Forms
---
<div>{{LearnSidebar}}</div>

<div>{{PreviousMenuNext("Learn/Server-side/Django/authentication_and_sessions", "Learn/Server-side/Django/Testing", "Learn/Server-side/Django")}}</div>

<p>在本教程中，我們將向您展示，如何在 Django 中使用 HTML 表單，特別是編寫表單以創建，更新和刪除模型實例的最簡單方法。作為本演示的一部分，我們將擴展 <a href="/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website">LocalLibrary </a>網站，以便圖書館員，可以使用我們自己的表單（而不是使用管理員應用程序）更新圖書，創建，更新和刪除作者。</p>

<table class="learn-box standard-table">
 <tbody>
  <tr>
   <th scope="row"> 前提:</th>
   <td>完成先前所有的教程, 包含 <a href="/en-US/docs/Learn/Server-side/Django/authentication_and_sessions">Django Tutorial Part 8: User authentication and permissions</a>.</td>
  </tr>
  <tr>
   <th scope="row">目的:</th>
   <td>了解如何製作表單來向用戶取得資訊並更新資料庫。了解<strong>通用類別表單編輯視圖 </strong>( generic class-based form editing views ) 能夠大幅簡化用於單一模型的表單製作。</td>
  </tr>
 </tbody>
</table>

<h2 id="概述">概述</h2>

<p><a href="/en-US/docs/Web/Guide/HTML/Forms">HTML表單</a>是網頁上的一組一個或多個字段/小組件，可用於從用戶收集信息以提交到服務器。 表單是一種用於收集用戶輸入的靈活機制，因為有合適的小部件可以輸入許多不同類型的數據，包括文本框，複選框，單選按鈕，日期選擇器等。表單也是與服務器共享數據的相對安全的方式， 因為它們允許我們在具有跨站點請求偽造保護的<code>POST</code> 請求中發送數據。</p>

<p>儘管到目前為止，本教程中尚未創建任何表單，但我們已經在Django Admin網站中遇到過這些表單-例如，下面的屏幕截圖顯示了一種用於編輯我們的<a href="/en-US/docs/Learn/Server-side/Django/Models">Book</a> 模型的表單，該表單由許多選擇列表和 文字編輯器。</p>

<p><img alt="Admin Site - Book Add" src="admin_book_add.png"></p>

<p>使用表單可能會很複雜！開發人員需要為表單編寫HTML，在服務器上（也可能在瀏覽器中）驗證並正確清理輸入的數據，使用錯誤消息重新發布表單以通知用戶任何無效字段，並在成功提交數據後處理數據，最後以某種方式回應用戶以表示成功。 Django表單通過提供一個框架使您能夠以編程方式定義表單及其字段，然後使用這些對像生成表單HTML代碼並處理許多驗證和用戶交互，從而完成了所有這些步驟中的大量工作。</p>

<p>在本教程中，我們將向您展示創建和使用表單的幾種方法，尤其是通用編輯表單視圖如何顯著減少創建表單來操縱表單所需的工作量。楷模。在此過程中，我們將擴展本地圖書館應用程序，方法是添加一個允許圖書館員續訂圖書的表格，並創建頁面以創建，編輯和刪除圖書和作者（複製上面顯示的表格的基本版本以編輯圖書） ）。</p>

<h2 id="HTML_表單">HTML 表單</h2>

<p>首先簡要介紹一下 <a href="/en-US/docs/Learn/HTML/Forms">HTML Forms</a>。 考慮一個簡單的 HTML 表單，其中有一個用於輸入某些“團隊”名稱的文本字段及其相關標籤：</p>

<p><img alt="Simple name field example in HTML form" src="form_example_name_field.png"></p>

<p>表單在HTML中定義為 <code>&lt;form&gt;...&lt;/form&gt;</code> 標記內元素的集合，其中至少包含<code>type="submit"</code>.的<code>input</code>元素。</p>

<pre class="brush: html notranslate">&lt;form action="/team_name_url/" method="post"&gt;
    &lt;label for="team_name"&gt;Enter name: &lt;/label&gt;
    &lt;input id="team_name" type="text" name="name_field" value="Default name for team."&gt;
    &lt;input type="submit" value="OK"&gt;
&lt;/form&gt;</pre>

<p>雖然這裡只有一個用於輸入團隊名稱的文本字段，但是表單可以具有任意數量的其他輸入元素及其關聯的標籤。字段的<code>type</code> 屬性定義將顯示哪種小部件。字段的 <code>name</code> 和<code>id</code> 用於標識JavaScript / CSS / HTML中的字段，而 <code>value</code>定義該字段在首次顯示時的初始值。匹配的團隊標籤是使用<code>label</code> 標籤指定的（請參見上面的“輸入名稱”），其中的  <code>for</code> 字段包含相關<code>input</code>的<code>id</code> 值。</p>

<p><code>submit</code> 輸入將顯示為一個按鈕（默認情況下），用戶可以按下該按鈕以將表單中所有其他輸入元素中的數據上載到服務器（在這種情況下，僅是<code>team_name</code>）。表單屬性定義用於發送數據的HTTP<code>method</code> 以及服務器上數據的目的地（<code>action</code>）：<br>
  </p>

<ul>
 <li><code>action</code>: 提交表單時，將數據發送到該資源/ URL進行處理。如果未設置（或設置為空字符串），則表單將被提交回當前頁面URL。</li>
 <li><code>method</code>: 用於發送數據的HTTP方法：post或get。
  <ul>
   <li>如果數據將導致服務器數據庫的更改，則應始終使用<code>POST</code> 方法，因為這樣可以使它更能抵抗跨站點的偽造請求攻擊。</li>
   <li><code>GET</code> 方法應僅用於不更改用戶數據的表單（例如搜索表單）。建議您在希望添加書籤或共享URL時使用。</li>
  </ul>
 </li>
</ul>

<p>服務器的角色是首先呈現初始表單狀態-包含空白字段，或預填充初始值。用戶按下“提交”按鈕後，服務器將從Web瀏覽器接收帶有值的表單數據，並且必須驗證信息。如果表單包含無效數據，則服務器應再次顯示該表單，這一次將在“有效”字段中顯示用戶輸入的數據，並顯示描述無效字段問題的消息。服務器收到包含所有有效表單數據的請求後，便可以執行適當的操作（例如，保存數據，返回搜索結果，上傳文件等），然後通知用戶。</p>

<p>可以想像，創建HTML，驗證返回的數據，在需要時使用錯誤報告重新顯示輸入的數據以及對有效數據執行所需的操作都需要花費大量精力才能“正確”。 Django通過刪除一些繁瑣且重複的代碼，使此操作變得更加容易！</p>

<h2 id="Django表單處理流程">Django表單處理流程</h2>

<p>Django的表單處理使用了我們在以前的教程中學到的所有相同技術（用於顯示有關模型的信息）：視圖獲取請求，執行所需的任何操作，包括從模型中讀取數據，然後生成並返回HTML頁面（ 從模板中，我們傳遞一個包含要顯示的數據的上下文）。 使事情變得更加複雜的是，服務器還需要能夠處理用戶提供的數據，並在出現任何錯誤時重新顯示頁面。</p>

<p>下面顯示了Django處理表單請求的過程流程圖，該流程圖從對包含表單的頁面的請求（以綠色顯示）開始。<br>
 <img alt="Updated form handling process doc." src="form_handling_-_standard.png"></p>

<p>根據上圖，Django表單處理的主要功能是：</p>

<ol>
 <li>在用戶第一次請求時顯示默認表單。
  <ul>
   <li>該表單可能包含空白字段（例如，如果您正在創建新記錄），或者可能會預先填充有初始值（例如，如果您正在更改記錄或具有有用的默認初始值）。</li>
   <li>由於此表單與任何用戶輸入的數據均不相關（儘管它可能具有初始值），因此在這一點上被稱為未綁定。</li>
  </ul>
 </li>
 <li>從提交請求中接收數據並將其綁定到表單。
  <ul>
   <li>將數據綁定到表單意味著當我們需要重新顯示表單時，用戶輸入的數據和任何錯誤均可用。</li>
  </ul>
 </li>
 <li>清理並驗證數據。
  <ul>
   <li>清理數據會對輸入執行清理操作（例如，刪除可能用於向服務器發送惡意內容的無效字符），並將其轉換為一致的Python類型。</li>
   <li>驗證會檢查該值是否適合該字段（例如，日期範圍正確，時間不要太短或太長等）</li>
  </ul>
 </li>
 <li>如果任何數據無效，則這次重新顯示該表單，其中包含用戶填充的所有值和問題字段的錯誤消息。</li>
 <li>如果所有數據均有效，請執行所需的操作（例如，保存數據，發送和發送電子郵件，返回搜索結果，上傳文件等）</li>
 <li>完成所有操作後，將用戶重定向到另一個頁面。</li>
</ol>

<p>Django提供了許多工具和方法來幫助您完成上述任務。 最基本的是 <code>Form</code>類，它簡化了表單HTML的生成和數據清除/驗證的過程。 在下一節中，我們將使用頁面的實際示例描述表單如何工作，以使圖書館員可以續訂書籍。</p>

<div class="note">
<p><strong>備註：</strong> 當我們討論Django的更多“高級”表單框架類時，了解<code>Form</code>的使用方式將對您有所幫助。</p>
</div>

<h2 id="使用表單和功能視圖續訂表單">使用表單和功能視圖續訂表單</h2>

<p>接下來，我們將添加一個頁面，以使圖書館員可以續借借來的書。 為此，我們將創建一個允許用戶輸入日期值的表單。 我們將從當前日期（正常藉閱期）起3週內為該字段提供初始值，並添加一些驗證以確保館員不能輸入過去的日期或將來的日期。 輸入有效日期後，我們會將其寫入當前記錄的<code>BookInstance.due_back</code> 字段中。</p>

<p>該示例將使用基於函數的視圖和<code>Form</code> 類。 以下各節說明表單的工作方式，以及您需要對正在進行的LocalLibrary項目進行的更改。</p>

<h3 id="Form">Form</h3>

<p><code>Form</code>類是Django表單處理系統的核心。 它指定表單中的字段，其佈局，顯示小部件，標籤，初始值，有效值，以及（一旦驗證）與無效字段關聯的錯誤消息。 該類還提供了使用預定義格式（表，列表等）在模板中呈現自身的方法，或用於獲取任何元素的值（啟用細粒度手動呈現）的方法。</p>

<h4 id="申報表格">申報表格</h4>

<p><code>Form</code> 的聲明語法與聲明<code>Model</code>的語法非常相似，並且具有相同的字段類型（和一些相似的參數）。 這是有道理的，因為在兩種情況下，我們都需要確保每個字段都處理正確的數據類型，被限制為有效數據並具有顯示/文檔描述。</p>

<p>要創建一個表單，我們導入<code>Form</code> 庫，從<code>Form</code> 類派生，並聲明表單的字段。 下面顯示了我們的圖書館圖書續訂表格的一個非常基本的表格類：</p>

<pre class="brush: python notranslate">from django import forms

class RenewBookForm(forms.Form):
    renewal_date = forms.DateField(help_text="Enter a date between now and 4 weeks (default 3).")
</pre>

<h4 id="Form_fields">Form fields</h4>

<p>In this case we have a single <code><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#datefield">DateField</a></code> for entering the renewal date that will render in HTML with a blank value, the default label "<em>Renewal date:</em>", and some helpful usage text: "<em>Enter a date between now and 4 weeks (default 3 weeks).</em>" As none of the other optional arguments are specified the field will accept dates using the <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#django.forms.DateField.input_formats">input_formats</a>: YYYY-MM-DD (2016-11-06), MM/DD/YYYY (02/26/2016), MM/DD/YY (10/25/16), and will be rendered using the default <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#widget">widget</a>: <a href="https://docs.djangoproject.com/en/2.0/ref/forms/widgets/#django.forms.DateInput">DateInput</a>.</p>

<p>There are many other types of form fields, which you will largely recognise from their similarity to the equivalent model field classes: <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#booleanfield"><code>BooleanField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#charfield"><code>CharField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#choicefield"><code>ChoiceField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#typedchoicefield"><code>TypedChoiceField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#datefield"><code>DateField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#datetimefield"><code>DateTimeField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#decimalfield"><code>DecimalField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#durationfield"><code>DurationField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#emailfield"><code>EmailField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#filefield"><code>FileField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#filepathfield"><code>FilePathField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#floatfield"><code>FloatField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#imagefield"><code>ImageField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#integerfield"><code>IntegerField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#genericipaddressfield"><code>GenericIPAddressField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#multiplechoicefield"><code>MultipleChoiceField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#typedmultiplechoicefield"><code>TypedMultipleChoiceField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#nullbooleanfield"><code>NullBooleanField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#regexfield"><code>RegexField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#slugfield"><code>SlugField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#timefield"><code>TimeField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#urlfield"><code>URLField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#uuidfield"><code>UUIDField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#combofield"><code>ComboField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#multivaluefield"><code>MultiValueField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#splitdatetimefield"><code>SplitDateTimeField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#modelmultiplechoicefield"><code>ModelMultipleChoiceField</code></a>, <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#modelchoicefield"><code>ModelChoiceField</code></a>​​​​.</p>

<p>The arguments that are common to most fields are listed below (these have sensible default values):</p>

<ul>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#required">required</a>: If <code>True</code>, the field may not be left blank or given a <code>None</code> value. Fields are required by default, so you would set <code>required=False</code> to allow blank values in the form.</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#label">label</a>: The label to use when rendering the field in HTML. If <a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#label">label</a> is not specified then Django would create one from the field name by capitalising the first letter and replacing underscores with spaces (e.g. <em>Renewal date</em>).</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#label-suffix">label_suffix</a>: By default a colon is displayed after the label (e.g. Renewal date<strong>:</strong>). This argument allows you to specify a different suffix containing other character(s).</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#initial">initial</a>: The initial value for the field when the form is displayed.</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#widget">widget</a>: The display widget to use.</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#help-text">help_text</a> (as seen in the example above): Additional text that can be displayed in forms to explain how to use the field.</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#error-messages">error_messages</a>: A list of error messages for the field. You can override these with your own messages if needed.</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#validators">validators</a>: A list of functions that will be called on the field when it is validated.</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#localize">localize</a>: Enables the localisation of form data input (see link for more information).</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/#disabled">disabled</a>: The field is displayed but its value cannot be edited if this is <code>True</code>. The default is <code>False</code>.</li>
</ul>

<h4 id="Validation">Validation</h4>

<p>Django provides numerous places where you can validate your data. The easiest way to validate a single field is to override the method <code>clean_<strong>&lt;fieldname&gt;</strong>()</code> for the field you want to check. So for example, we can validate that entered <code>renewal_date</code> values are between now and 4 weeks by implementing <code>clean_<strong>renewal_date</strong>() </code>as shown below.</p>

<pre class="brush: python notranslate">from django import forms

<strong>from django.core.exceptions import ValidationError
from django.utils.translation import ugettext_lazy as _
import datetime #for checking renewal date range.
</strong>
class RenewBookForm(forms.Form):
    renewal_date = forms.DateField(help_text="Enter a date between now and 4 weeks (default 3).")

<strong>    def clean_renewal_date(self):
        data = self.cleaned_data['renewal_date']

        #Check date is not in past.
        if data &lt; datetime.date.today():
            raise ValidationError(_('Invalid date - renewal in past'))

        #Check date is in range librarian allowed to change (+4 weeks).
        if data &gt; datetime.date.today() + datetime.timedelta(weeks=4):
            raise ValidationError(_('Invalid date - renewal more than 4 weeks ahead'))

        # Remember to always return the cleaned data.
        return data</strong></pre>

<p>There are two important things to note. The first is that we get our data using <code>self.cleaned_data['renewal_date']</code> and that we return this data whether or not we change it at the end of the function. This step gets us the data "cleaned" and sanitised of potentially unsafe input using the default validators, and converted into the correct standard type for the data (in this case a Python <code>datetime.datetime</code> object).</p>

<p>The second point is that if a value falls outside our range we raise a <code>ValidationError</code>, specifying the error text that we want to display in the form if an invalid value is entered. The example above also wraps this text in one of <a href="https://docs.djangoproject.com/en/2.0/topics/i18n/translation/">Django's translation functions</a> <code>ugettext_lazy()</code> (imported as <code>_()</code>), which is good practice if you want to translate your site later.</p>

<div class="note">
<p><strong>備註：</strong> There are numerious other methods and examples for validating forms in <a href="https://docs.djangoproject.com/en/2.0/ref/forms/validation/">Form and field validation</a> (Django docs). For example, in cases where you have multiple fields that depend on each other, you can override the <a href="https://docs.djangoproject.com/en/2.0/ref/forms/api/#django.forms.Form.clean">Form.clean()</a> function and again raise a <code>ValidationError</code>.</p>
</div>

<p>That's all we need for the form in this example!</p>

<h4 id="Copy_the_Form">Copy the Form</h4>

<p>Create and open the file <strong>locallibrary/catalog/forms.py</strong> and copy the entire code listing from the previous block into it.</p>

<h3 id="URL_Configuration">URL Configuration</h3>

<p>Before we create our view, let's add a URL configuration for the <em>renew-books</em> page. Copy the following configuration to the bottom of <strong>locallibrary/catalog/urls.py</strong>.</p>

<pre class="brush: python notranslate">urlpatterns += [
    path('book/&lt;uuid:pk&gt;/renew/', views.renew_book_librarian, name='renew-book-librarian'),
]</pre>

<p>The URL configuration will redirect URLs with the format <strong>/catalog/book/<em>&lt;bookinstance id&gt;</em>/renew/</strong> to the function named <code>renew_book_librarian()</code> in <strong>views.py</strong>, and send the <code>BookInstance</code> id as the parameter named <code>pk</code>. The pattern only matches if <code>pk</code> is a correctly formatted <code>uuid</code>.</p>

<div class="note">
<p><strong>備註：</strong> We can name our captured URL data "<code>pk</code>" anything we like, because we have complete control over the view function (we're not using a generic detail view class that expects parameters with a certain name). However <code>pk</code>, short for "primary key", is a reasonable convention to use!</p>
</div>

<h3 id="View">View</h3>

<p>As discussed in the <a href="#django_form_handling_process">Django form handling process</a> above, the view has to render the default form when it is first called and then either re-render it with error messages if the data is invalid, or process the data and redirect to a new page if the data is valid. In order to perform these different actions, the view has to be able to know whether it is being called for the first time to render the default form, or a subsequent time to validate data. </p>

<p>For forms that use a <code>POST</code> request to submit information to the server, the most common pattern is for the view to test against the <code>POST</code> request type (<code>if request.method == 'POST':</code>) to identify form validation requests and <code>GET</code> (using an <code>else</code> condition) to identify the initial form creation request. If you want to submit your data using a <code>GET</code> request then a typical approach for identifying whether this is the first or subsequent view invocation is to read the form data (e.g. to read a hidden value in the form).</p>

<p>The book renewal process will be writing to our database, so by convention we use the <code>POST</code> request approach. The code fragment below shows the (very standard) pattern for this sort of function view. </p>

<pre class="brush: python notranslate">from django.shortcuts import get_object_or_404
from django.http import HttpResponseRedirect
from django.urls import reverse
import datetime

from .forms import RenewBookForm

def renew_book_librarian(request, pk):
    book_inst=get_object_or_404(BookInstance, pk = pk)

    # If this is a POST request then process the Form data
<strong>    if request.method == 'POST':</strong>

        # Create a form instance and populate it with data from the request (binding):
        form = RenewBookForm(request.POST)

        # Check if the form is valid:
        <strong>if form.is_valid():</strong>
            # process the data in form.cleaned_data as required (here we just write it to the model due_back field)
            book_inst.due_back = form.cleaned_data['renewal_date']
            book_inst.save()

            # redirect to a new URL:
            return HttpResponseRedirect(reverse('all-borrowed') )

    # If this is a GET (or any other method) create the default form.
<strong>    else:</strong>
        proposed_renewal_date = datetime.date.today() + datetime.timedelta(weeks=3)
        form = RenewBookForm(initial={'renewal_date': proposed_renewal_date,})

    return render(request, 'catalog/book_renew_librarian.html', {'form': form, 'bookinst':book_inst})</pre>

<p>First we import our form (<code>RenewBookForm</code>) and a number of other useful objects/methods used in the body of the view function:</p>

<ul>
 <li><code><a href="https://docs.djangoproject.com/en/2.0/topics/http/shortcuts/#get-object-or-404">get_object_or_404()</a></code>: Returns a specified object from a model based on its primary key value, and raises an <code>Http404</code> exception (not found) if the record does not exist. </li>
 <li><code><a href="https://docs.djangoproject.com/en/2.0/ref/request-response/#django.http.HttpResponseRedirect">HttpResponseRedirect</a></code>: This creates a redirect to a specified URL (HTTP status code 302). </li>
 <li><code><a href="https://docs.djangoproject.com/en/2.0/ref/urlresolvers/#django.urls.reverse">reverse()</a></code>: This generates a URL from a URL configuration name and a set of arguments. It is the Python equivalent of the <code>url</code> tag that we've been using in our templates.</li>
 <li><code><a href="https://docs.python.org/3/library/datetime.html">datetime</a></code>: A Python library for manipulating dates and times. </li>
</ul>

<p>In the view we first use the <code>pk</code> argument in <code>get_object_or_404()</code> to get the current <code>BookInstance</code> (if this does not exist, the view will immediately exit and the page will display a "not found" error). If this is <em>not </em>a <code>POST</code> request (handled by the <code>else</code> clause) then we create the default form passing in an <code>initial</code> value for the <code>renewal_date</code> field (as shown in bold below, this is 3 weeks from the current date). </p>

<pre class="brush: python notranslate">    book_inst=get_object_or_404(BookInstance, pk = pk)

    # If this is a GET (or any other method) create the default form
    <strong>else:</strong>
        proposed_renewal_date = datetime.date.today() + datetime.timedelta(<strong>weeks=3</strong>)
        <strong>form = RenewBookForm(initial={'</strong>renewal_date<strong>': </strong>proposed_renewal_date<strong>,})</strong>

    return render(request, 'catalog/book_renew_librarian.html', {'form': form, 'bookinst':book_inst})</pre>

<p>After creating the form, we call <code>render()</code> to create the HTML page, specifying the template and a context that contains our form. In this case the context also contains our <code>BookInstance</code>, which we'll use in the template to provide information about the book we're renewing.</p>

<p>If however this is a <code>POST</code> request, then we create our <code>form</code> object and populate it with data from the request. This process is called "binding" and allows us to validate the form. We then check if the form is valid, which runs all the validation code on all of the fields — including both the generic code to check that our date field is actually a valid date and our specific form's <code>clean_renewal_date()</code> function to check the date is in the right range. </p>

<pre class="brush: python notranslate">    book_inst=get_object_or_404(BookInstance, pk = pk)

    # If this is a POST request then process the Form data
    if request.method == 'POST':

        # Create a form instance and populate it with data from the request (binding):
<strong>        form = RenewBookForm(request.POST)</strong>

        # Check if the form is valid:
        if form.is_valid():
            # process the data in form.cleaned_data as required (here we just write it to the model due_back field)
            book_inst.due_back = form.cleaned_data['renewal_date']
            book_inst.save()

            # redirect to a new URL:
            return HttpResponseRedirect(reverse('all-borrowed') )

    return render(request, 'catalog/book_renew_librarian.html', {'form': form, 'bookinst':book_inst})</pre>

<p>If the form is not valid we call <code>render()</code> again, but this time the form value passed in the context will include error messages. </p>

<p>If the form is valid, then we can start to use the data, accessing it through the <code>form.cleaned_data</code> attribute (e.g. <code>data = form.cleaned_data['renewal_date']</code>). Here we just save the data into the <code>due_back</code> value of the associated <code>BookInstance</code> object.</p>

<div class="warning">
<p><strong>警告：</strong> While you can also access the form data directly through the request (for example <code>request.POST['renewal_date']</code> or <code>request.GET['renewal_date']</code> (if using a GET request) this is NOT recommended. The cleaned data is sanitised, validated, and converted into Python-friendly types.</p>
</div>

<p>The final step in the form-handling part of the view is to redirect to another page, usually a "success" page. In this case we use <code>HttpResponseRedirect</code> and <code>reverse()</code> to redirect to the view named <code>'all-borrowed'</code> (this was created as the "challenge" in <a href="/en-US/docs/Learn/Server-side/Django/authentication_and_sessions#Challenge_yourself">Django Tutorial Part 8: User authentication and permissions</a>). If you didn't create that page consider redirecting to the home page at URL '/').</p>

<p>That's everything needed for the form handling itself, but we still need to restrict access to the view to librarians. We should probably create a new permission in <code>BookInstance</code> ("<code>can_renew</code>"), but to keep things simple here we just use the <code>@permission_required</code> function decorator with our existing <code>can_mark_returned</code> permission.</p>

<p>The final view is therefore as shown below. Please copy this into the bottom of <strong>locallibrary/catalog/views.py</strong>.</p>

<pre class="notranslate"><strong>from django.contrib.auth.decorators import permission_required</strong>

from django.shortcuts import get_object_or_404
from django.http import HttpResponseRedirect
from django.urls import reverse
import datetime

from .forms import RenewBookForm

<strong>@permission_required('catalog.<code>can_mark_returned</code>')</strong>
def renew_book_librarian(request, pk):
    """
    View function for renewing a specific BookInstance by librarian
    """
    book_inst=get_object_or_404(BookInstance, pk = pk)

    # If this is a POST request then process the Form data
    if request.method == 'POST':

        # Create a form instance and populate it with data from the request (binding):
        form = RenewBookForm(request.POST)

        # Check if the form is valid:
        if form.is_valid():
            # process the data in form.cleaned_data as required (here we just write it to the model due_back field)
            book_inst.due_back = form.cleaned_data['renewal_date']
            book_inst.save()

            # redirect to a new URL:
            return HttpResponseRedirect(reverse('all-borrowed') )

    # If this is a GET (or any other method) create the default form.
    else:
        proposed_renewal_date = datetime.date.today() + datetime.timedelta(weeks=3)
        form = RenewBookForm(initial={'renewal_date': proposed_renewal_date,})

    return render(request, 'catalog/book_renew_librarian.html', {'form': form, 'bookinst':book_inst})
</pre>

<h3 id="The_template">The template</h3>

<p>Create the template referenced in the view (<strong>/catalog/templates/catalog/book_renew_librarian.html</strong>) and copy the code below into it:</p>

<pre class="brush: html notranslate">{% extends "base_generic.html" %}
{% block content %}

    &lt;h1&gt;Renew: \{{bookinst.book.title}}&lt;/h1&gt;
    &lt;p&gt;Borrower: \{{bookinst.borrower}}&lt;/p&gt;
    &lt;p{% if bookinst.is_overdue %} class="text-danger"{% endif %}&gt;Due date: \{{bookinst.due_back}}&lt;/p&gt;

<strong>    &lt;form action="" method="post"&gt;
        {% csrf_token %}
        &lt;table&gt;
        \{{ form }}
        &lt;/table&gt;
        &lt;input type="submit" value="Submit" /&gt;
    &lt;/form&gt;</strong>

{% endblock %}</pre>

<p>Most of this will be completely familiar from previous tutorials. We extend the base template and then redefine the content block. We are able to reference <code>\{{bookinst}}</code> (and its variables) because it was passed into the context object in the <code>render()</code> function, and we use these to list the book title, borrower and the original due date.</p>

<p>The form code is relatively simple. First we declare the <code>form</code> tags, specifying where the form is to be submitted (<code>action</code>) and the <code>method</code> for submitting the data (in this case an "HTTP POST") — if you recall the <a href="#HTML_forms">HTML Forms</a> overview at the top of the page, an empty <code>action</code> as shown, means that the form data will be posted back to the current URL of the page (which is what we want!). Inside the tags we define the <code>submit</code> input, which a user can press to submit the data. The <code>{% csrf_token %}</code> added just inside the form tags is part of Django's cross-site forgery protection.</p>

<div class="note">
<p><strong>備註：</strong> Add the <code>{% csrf_token %}</code> to every Django template you create that uses <code>POST</code> to submit data. This will reduce the chance of forms being hijacked by malicious users.</p>
</div>

<p>All that's left is the <code>\{{form}}</code> template variable, which we passed to the template in the context dictionary. Perhaps unsurprisingly, when used as shown this provides the default rendering of all the form fields, including their labels, widgets, and help text — the rendering is as shown below:</p>

<pre class="brush: html notranslate">&lt;tr&gt;
  &lt;th&gt;&lt;label for="id_renewal_date"&gt;Renewal date:&lt;/label&gt;&lt;/th&gt;
  &lt;td&gt;
    &lt;input id="id_renewal_date" name="renewal_date" type="text" value="2016-11-08" required /&gt;
    &lt;br /&gt;
    &lt;span class="helptext"&gt;Enter date between now and 4 weeks (default 3 weeks).&lt;/span&gt;
  &lt;/td&gt;
&lt;/tr&gt;
</pre>

<div class="note">
<p><strong>備註：</strong> It is perhaps not obvious because we only have one field, but by default every field is defined in its own table row (which is why the variable is inside <code>table </code>tags above).​​​​​​ This same rendering is provided if you reference the template variable <code>\{{ form.as_table }}</code>.</p>
</div>

<p>If you were to enter an invalid date, you'd additionally get a list of the errors rendered in the page (shown in bold below).</p>

<pre class="brush: html notranslate">&lt;tr&gt;
  &lt;th&gt;&lt;label for="id_renewal_date"&gt;Renewal date:&lt;/label&gt;&lt;/th&gt;
   &lt;td&gt;
<strong>      &lt;ul class="errorlist"&gt;
        &lt;li&gt;Invalid date - renewal in past&lt;/li&gt;
      &lt;/ul&gt;</strong>
      &lt;input id="id_renewal_date" name="renewal_date" type="text" value="2015-11-08" required /&gt;
      &lt;br /&gt;
      &lt;span class="helptext"&gt;Enter date between now and 4 weeks (default 3 weeks).&lt;/span&gt;
    &lt;/td&gt;
&lt;/tr&gt;</pre>

<h4 id="Other_ways_of_using_form_template_variable">Other ways of using form template variable</h4>

<p>Using <code>\{{form}}</code> as shown above, each field is rendered as a table row. You can also render each field as a list item (using <code>\{{form.as_ul}}</code> ) or as a paragraph (using <code>\{{form.as_p}}</code>).</p>

<p>What is even more cool is that you can have complete control over the rendering of each part of the form, by indexing its properties using dot notation. So for example we can access a number of separate items for our <code>renewal_date</code> field:</p>

<ul>
 <li><code>\{{form.renewal_date}}:</code> The whole field.</li>
 <li><code>\{{form.renewal_date.errors}}</code>: The list of errors.</li>
 <li><code>\{{form.renewal_date.id_for_label}}</code>: The id of the label.</li>
 <li><code>\{{form.renewal_date.help_text}}</code>: The field help text.</li>
 <li>etc!</li>
</ul>

<p>For more examples of how to manually render forms in templates and dynamically loop over template fields, see <a href="https://docs.djangoproject.com/en/2.0/topics/forms/#rendering-fields-manually">Working with forms &gt; Rendering fields manually</a> (Django docs).</p>

<h3 id="Testing_the_page">Testing the page</h3>

<p>If you accepted the "challenge" in <a href="/en-US/docs/Learn/Server-side/Django/authentication_and_sessions#Challenge_yourself">Django Tutorial Part 8: User authentication and permissions</a> you'll have a list of all books on loan in the library, which is only visible to library staff. We can add a link to our renew page next to each item using the template code below.</p>

<pre class="brush: html notranslate">{% if perms.catalog.can_mark_returned %}- &lt;a href="{% url 'renew-book-librarian' bookinst.id %}"&gt;Renew&lt;/a&gt;  {% endif %}</pre>

<div class="note">
<p><strong>備註：</strong> Remember that your test login will need to have the permission "<code>catalog.can_mark_returned</code>" in order to access the renew book page (perhaps use your superuser account).</p>
</div>

<p>You can alternatively manually construct a test URL like this — <a href="http://127.0.0.1:8000/catalog/book/&lt;bookinstance id>/renew/">http://127.0.0.1:8000/catalog/book/<em>&lt;bookinstance_id&gt;</em>/renew/</a> (a valid bookinstance id can be obtained by navigating to a book detail page in your library, and copying the <code>id</code> field).</p>

<h3 id="What_does_it_look_like">What does it look like?</h3>

<p>If you are successful, the default form will look like this:</p>

<p><img src="forms_example_renew_default.png"></p>

<p>The form with an invalid value entered, will look like this:</p>

<p><img src="forms_example_renew_invalid.png"></p>

<p>The list of all books with renew links will look like this:</p>

<p><img src="forms_example_renew_allbooks.png"></p>

<h2 id="ModelForms">ModelForms</h2>

<p>Creating a <code>Form</code> class using the approach described above is very flexible, allowing you to create whatever sort of form page you like and associate it with any model or models.</p>

<p>However if you just need a form to map the fields of a <em>single</em> model then your model will already define most of the information that you need in your form: fields, labels, help text, etc. Rather than recreating the model definitions in your form, it is easier to use the <a href="https://docs.djangoproject.com/en/2.0/topics/forms/modelforms/">ModelForm</a> helper class to create the form from your model. This <code>ModelForm</code> can then be used within your views in exactly the same way as an ordinary <code>Form</code>.</p>

<p>A basic <code>ModelForm</code> containing the same field as our original <code>RenewBookForm</code> is shown below. All you need to do to create the form is add <code>class Meta</code> with the associated <code>model</code> (<code>BookInstance</code>) and a list of the model <code>fields</code> to include in the form (you can include all fields using <code>fields = '__all__'</code>, or you can use <code>exclude</code> (instead of <code>fields</code>) to specify the fields <em>not </em>to include from the model).</p>

<pre class="brush: python notranslate">from django.forms import ModelForm
from .models import BookInstance

class RenewBookModelForm(ModelForm):
<strong>    class Meta:
        model = BookInstance
        fields = ['due_back',]</strong>
</pre>

<div class="note">
<p><strong>備註：</strong> This might not look like all that much simpler than just using a <code>Form</code> (and it isn't in this case, because we just have one field). However if you have a lot of fields, it can reduce the amount of code quite significantly!</p>
</div>

<p>The rest of the information comes from the model field definitions (e.g. labels, widgets, help text, error messages). If these aren't quite right, then we can override them in our <code>class Meta</code>, specifying a dictionary containing the field to change and its new value. For example, in this form we might want a label for our field of "<em>Renewal date</em>" (rather than the default based on the field name: <em>Due date</em>), and we also want our help text to be specific to this use case. The <code>Meta</code> below shows you how to override these fields, and you can similarly set <code>widgets</code> and <code>error_messages</code> if the defaults aren't sufficient.</p>

<pre class="brush: python notranslate">class Meta:
    model = BookInstance
    fields = ['due_back',]
<strong>    labels = { 'due_back': _('Renewal date'), }
    help_texts = { 'due_back': _('Enter a date between now and 4 weeks (default 3).'), } </strong>
</pre>

<p>To add validation you can use the same approach as for a normal <code>Form</code> — you define a function named <code>clean_<em>field_name</em>()</code> and raise <code>ValidationError</code> exceptions for invalid values. The only difference with respect to our original form is that the model field is named <code>due_back</code> and not "<code>renewal_date</code>".</p>

<pre class="brush: python notranslate">from django.forms import ModelForm
from .models import BookInstance

class RenewBookModelForm(ModelForm):
<strong>    def clean_due_back(self):
       data = self.cleaned_data['due_back']

       #Check date is not in past.
       if data &lt; datetime.date.today():
           raise ValidationError(_('Invalid date - renewal in past'))

       #Check date is in range librarian allowed to change (+4 weeks)
       if data &gt; datetime.date.today() + datetime.timedelta(weeks=4):
           raise ValidationError(_('Invalid date - renewal more than 4 weeks ahead'))

       # Remember to always return the cleaned data.
       return data
</strong>
    class Meta:
        model = BookInstance
        fields = ['due_back',]
        labels = { 'due_back': _('Renewal date'), }
        help_texts = { 'due_back': _('Enter a date between now and 4 weeks (default 3).'), }
</pre>

<p>The class <code>RenewBookModelForm</code> below is now functionally equivalent to our original <code>RenewBookForm</code>. You could import and use it wherever you currently use <code>RenewBookForm</code>.</p>

<h2 id="Generic_editing_views">Generic editing views</h2>

<p>The form handling algorithm we used in our function view example above represents an extremely common pattern in form editing views. Django abstracts much of this "boilerplate" for you, by creating <a href="https://docs.djangoproject.com/en/2.0/ref/class-based-views/generic-editing/">generic editing views</a> for creating, editing, and deleting views based on models. Not only do these handle the "view" behaviour, but they automatically create the form class (a <code>ModelForm</code>) for you from the model.</p>

<div class="note">
<p><strong>備註：</strong> In addition to the editing views described here, there is also a <a href="https://docs.djangoproject.com/en/2.0/ref/class-based-views/generic-editing/#formview">FormView</a> class, which lies somewhere between our function view and the other generic views in terms of "flexibility" vs "coding effort". Using <code>FormView</code> you still need to create your <code>Form</code>, but you don't have to implement all of the standard form-handling pattern. Instead you just have to provide an implementation of the function that will be called once the submitted is known to be be valid.</p>
</div>

<p>In this section we're going to use generic editing views to create pages to add functionality to create, edit, and delete <code>Author</code> records from our library — effectively providing a basic reimplementation of parts of the Admin site (this could be useful if you need to offer admin functionality in a more flexible way that can be provided by the admin site).</p>

<h3 id="Views">Views</h3>

<p>Open the views file (<strong>locallibrary/catalog/views.py</strong>) and append the following code block to the bottom of it:</p>

<pre class="brush: python notranslate">from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Author

class AuthorCreate(CreateView):
    model = Author
    fields = '__all__'
    initial={'date_of_death':'05/01/2018',}

class AuthorUpdate(UpdateView):
    model = Author
    fields = ['first_name','last_name','date_of_birth','date_of_death']

class AuthorDelete(DeleteView):
    model = Author
    success_url = reverse_lazy('authors')</pre>

<p>As you can see, to create the views you need to derive from <code>CreateView</code>, <code>UpdateView</code>, and <code>DeleteView</code> (respectively) and then define the associated model.</p>

<p>For the "create" and "update" cases you also need to specify the fields to display in the form (using in same syntax as for <code>ModelForm</code>). In this case we show both the syntax to display "all" fields, and how you can list them individually. You can also specify initial values for each of the fields using a dictionary of <em>field_name</em>/<em>value</em> pairs (here we arbitrarily set the date of death for demonstration purposes — you might want to remove that!). By default these views will redirect on success to a page displaying the newly created/edited model item, which in our case will be the author detail view we created in a previous tutorial. You can specify an alternative redirect location by explicitly declaring parameter <code>success_url</code> (as done for the <code>AuthorDelete</code> class).</p>

<p>The <code>AuthorDelete</code> class doesn't need to display any of the fields, so these don't need to be specified. You do however need to specify the <code>success_url</code>, because there is no obvious default value for Django to use. In this case we use the <code><a href="https://docs.djangoproject.com/en/2.0/ref/urlresolvers/#reverse-lazy">reverse_lazy()</a></code> function to redirect to our author list after an author has been deleted — <code>reverse_lazy()</code> is a lazily executed version of <code>reverse()</code>, used here because we're providing a URL to a class-based view attribute.</p>

<h3 id="Templates">Templates</h3>

<p>The "create" and "update" views use the same template by default, which will be named after your model: <em>model_name</em><strong>_form.html</strong> (you can change the suffix to something other than <strong>_form</strong> using the <code>template_name_suffix</code> field in your view, e.g. <code>template_name_suffix = '_other_suffix'</code>)</p>

<p>Create the template file <strong>locallibrary/catalog/templates/catalog/author_form.html</strong> and copy in the text below.</p>

<pre class="brush: html notranslate">{% extends "base_generic.html" %}

{% block content %}

&lt;form action="" method="post"&gt;
    {% csrf_token %}
    &lt;table&gt;
    \{{ form.as_table }}
    &lt;/table&gt;
    &lt;input type="submit" value="Submit" /&gt;

&lt;/form&gt;
{% endblock %}</pre>

<p>This is similar to our previous forms, and renders the fields using a table. Note also how again we declare the <code>{% csrf_token %}</code> to ensure that our forms are resistant to CSRF attacks.</p>

<p>The "delete" view expects to find a template named with the format <em>model_name</em><strong>_confirm_delete.html</strong> (again, you can change the suffix using <code>template_name_suffix</code> in your view). Create the template file <strong>locallibrary/catalog/templates/catalog/author_confirm_delete</strong><strong>.html</strong> and copy in the text below.</p>

<pre class="brush: html notranslate">{% extends "base_generic.html" %}

{% block content %}

&lt;h1&gt;Delete Author&lt;/h1&gt;

&lt;p&gt;Are you sure you want to delete the author: \{{ author }}?&lt;/p&gt;

&lt;form action="" method="POST"&gt;
  {% csrf_token %}
  &lt;input type="submit" action="" value="Yes, delete." /&gt;
&lt;/form&gt;

{% endblock %}
</pre>

<h3 id="URL_configurations">URL configurations</h3>

<p>Open your URL configuration file (<strong>locallibrary/catalog/urls.py</strong>) and add the following configuration to the bottom of the file:</p>

<pre class="brush: python notranslate">urlpatterns += [
    path('author/create/', views.AuthorCreate.as_view(), name='author_create'),
    path('author/&lt;int:pk&gt;/update/', views.AuthorUpdate.as_view(), name='author_update'),
    path('author/&lt;int:pk&gt;/delete/', views.AuthorDelete.as_view(), name='author_delete'),
]</pre>

<p>There is nothing particularly new here! You can see that the views are classes, and must hence be called via <code>.as_view()</code>, and you should be able to recognise the URL patterns in each case. We must use <code>pk</code> as the name for our captured primary key value, as this is the parameter name expected by the view classes.</p>

<p>The author create, update, and delete pages are now ready to test (we won't bother hooking them into the site sidebar in this case, although you can do so if you wish).</p>

<div class="note">
<p><strong>備註：</strong> Observant users will have noticed that we didn't do anything to prevent unauthorised users from accessing the pages! We leave that as an exercise for you (hint: you could use the <code>PermissionRequiredMixin</code> and either create a new permission or reuse our <code>can_mark_returned</code> permission).</p>
</div>

<h3 id="Testing_the_page_2">Testing the page</h3>

<p>First login to the site with an account that has whatever permissions you decided are needed to access the author editing pages.</p>

<p>Then navigate to the author create page: <a href="http://127.0.0.1:8000/catalog/author/create/">http://127.0.0.1:8000/catalog/author/create/</a>, which should look like the screenshot below.</p>

<p><img alt="Form Example: Create Author" src="forms_example_create_author.png"></p>

<p>Enter values for the fields and then press <strong>Submit</strong> to save the author record. You should now be taken to a detail view for your new author, with a URL of something like <em>http://127.0.0.1:8000/catalog/author/10</em>.</p>

<p>You can test editing records by appending <em>/update/</em> to the end of the detail view URL (e.g. <em>http://127.0.0.1:8000/catalog/author/10/update/</em>) — we don't show a screenshot, because it looks just like the "create" page!</p>

<p>Last of all we can delete the page, by appending delete to the end of the author detail-view URL (e.g. <em>http://127.0.0.1:8000/catalog/author/10/delete/</em>). Django should display the delete page shown below. Press <strong>Yes, delete.</strong> to remove the record and be taken to the list of all authors.</p>

<p><img src="forms_example_delete_author.png"></p>

<h2 id="Challenge_yourself">Challenge yourself</h2>

<p>Create some forms to create, edit and delete <code>Book</code> records. You can use exactly the same structure as for <code>Authors</code>. If your <strong>book_form.html</strong> template is just a copy-renamed version of the <strong>author_form.html</strong> template, then the new "create book" page will look like the screenshot below:</p>

<p><img src="forms_example_create_book.png"></p>

<ul>
</ul>

<h2 id="Summary">Summary</h2>

<p>Creating and handling forms can be a complicated process! Django makes it much easier by providing programmatic mechanisms to declare, render and validate forms. Furthermore, Django provides generic form editing views that can do <em>almost all</em> the work to define pages that can create, edit, and delete records associated with a single model instance.</p>

<p>There is a lot more that can be done with forms (check out our See also list below), but you should now understand how to add basic forms and form-handling code to your own websites.</p>

<h2 id="See_also">See also</h2>

<ul>
 <li><a href="https://docs.djangoproject.com/en/2.0/topics/forms/">Working with forms</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/intro/tutorial04/#write-a-simple-form">Writing your first Django app, part 4 &gt; Writing a simple form</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/api/">The Forms API</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/fields/">Form fields</a> (Django docs) </li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/forms/validation/">Form and field validation</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/topics/class-based-views/generic-editing/">Form handling with class-based views</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/topics/forms/modelforms/">Creating forms from models</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/ref/class-based-views/generic-editing/">Generic editing views</a> (Django docs)</li>
</ul>

<p>{{PreviousMenuNext("Learn/Server-side/Django/authentication_and_sessions", "Learn/Server-side/Django/Testing", "Learn/Server-side/Django")}}</p>

<h2 id="In_this_module">In this module</h2>

<ul>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Introduction">Django introduction</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/development_environment">Setting up a Django development environment</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website">Django Tutorial: The Local Library website</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/skeleton_website">Django Tutorial Part 2: Creating a skeleton website</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Models">Django Tutorial Part 3: Using models</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Admin_site">Django Tutorial Part 4: Django admin site</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Home_page">Django Tutorial Part 5: Creating our home page</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Generic_views">Django Tutorial Part 6: Generic list and detail views</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Sessions">Django Tutorial Part 7: Sessions framework</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Authentication">Django Tutorial Part 8: User authentication and permissions</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Forms">Django Tutorial Part 9: Working with forms</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Testing">Django Tutorial Part 10: Testing a Django web application</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Django Tutorial Part 11: Deploying Django to production</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/web_application_security">Django web application security</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/django_assessment_blog">DIY Django mini blog</a></li>
</ul>