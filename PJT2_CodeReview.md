# 부스트코스 Android 학습 내용<br> 
** 학습한 내용 중 한번 더 정리하고 싶은 부분입니다.**<br>

### ◆ 인플레이션
+ <b>XML 레이아웃에 정의된 내용이 "메모리에 객체화"되는 과정</b><br>
( 레이아웃 정의  -->  메모리 로딩 -->  화면 )

+ <b>setContentView</b> 메소드에서 XML 레이아웃 파일 연결<br>
이때, 인플레이션 과정은 안드로이드 시스템에서 실행함

+ <b>전체 화면 중에서 일부분만 차지하는 화면 구성요소들은 LayoutInflater 클래스를 사용해야 한다.</b>

+ LayoutInflater inflater = (LayoutInflater) <b>getSystemService</b>(Context.LAYOUT_INFLATER_SERVICE);<br>
inflater.<b>inflate</b>(R.layout.sub1, container, true);<br><br>

### ◆ 리스트뷰
+ 대표적인 <b>선택 위젯</b> : 리스트뷰, 스피너, 그리드뷰, 갤러리

+ 사용방법
1) <b>아이템을 위한 xml 레이아웃</b> 정의하기
2) <b>아이템을 위한 뷰</b> 정의하기
3) 데이터 관리 역할을 하는 <b>어댑터 클래스</b>를 만들고 그 안에 각 아이템으로 표시할 뷰를 리턴하는 <b>getView()</b> 메소드 정의하기<br>
class <b>SingerAdapter</b> extends BaseAdapter {<br>
  &nbsp;&nbsp;&nbsp;&nbsp;ArrayList<SingerItem> items = new ArrayList<>();<br><br>
  &nbsp;&nbsp;&nbsp;&nbsp;public int <b>getCount()</b> { return items.size(); }<br>
  &nbsp;&nbsp;&nbsp;&nbsp;public Object <b>getItem(int position)</b> { return items.get(position); }<br>
  &nbsp;&nbsp;&nbsp;&nbsp;public long <b>getItemId(int position)</b> { return position; }<br>
  &nbsp;&nbsp;&nbsp;&nbsp;public View <b>getView(int position, View convertView, ViewGroup parent)</b> { <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SingerItemView view = new SingerItemView(getApplicationContext());<br><br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SingerItem item = items.get(position);<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;view.setName(item.getName());<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;view.setMobile(item.getMobile());<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return view;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>
}<br>
4) 화면에 보여줄 <b>리스트뷰</b>를 만들고 그 안에 데이터가 선택되었을 때 호출될 리스너 객체 정의하기<br>
  <b>listView.setOnItemClickListener()</b><br><br>

### ◆ 스피너
+ 콤보박스

+ 사용방법<br>
<b>Spinner</b> spinner = (Spinner) findViewById(R.id.spinner);<br>
<b>ArrayAdapter<String> adapter = new ArrayAdapter<String>(this, android.R.layout.simple_spinner_item, items);</b><br>
  adapter.<b>setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);</b><br>
spinner.setAdapter(adapter);<br><br>
spinner.<b>setOnItemSelectedListener</b>(...)
<br><br><br>

# PJT2. 좋아요와 한줄평 리스트 Code Review
<table>
  <tr><td><img src="https://user-images.githubusercontent.com/25261296/61945965-4e219380-afdc-11e9-9fb3-b50ee8c74929.png" width="250"></td>
      <td><img src="https://user-images.githubusercontent.com/25261296/61956662-9484eb80-aff8-11e9-8ae8-21225cdfe825.png" width="250"></td>
      <td><img src="https://user-images.githubusercontent.com/25261296/61946054-87f29a00-afdc-11e9-8715-0c820a5fbdfa.png" width="250"></td></tr>
</table>

### ◆ Q&A
<b>Question</b><br>
▷ "context"에 MainActivity.this를 넣는 것과 this만 넣는 것의 차이가 있나요?<br><br>
<b>Answer</b><br>
▷ this의 경우 자기 자신을 의미하기 때문에 MainActivity.this와 MainActivity의 this는 동일한 Context 입니다. equal 함수를 통해 MainActivity.this와 this가 같은지 비교할 수 있습니다.
<br><br>

### ◆ Advice 1
> 문구 리소스의 경우 <b>다국어 처리와 재사용</b>을 위해 <b>string.xml에 정의</b>하여 사용합니다. string 뿐만 아니라 color, dimen 등 하드 코딩되는 값들을 리소스에 정의해서 사용하세요.
<br>

### ◆ Advice 2
> 하나의 함수에서는 한가지 일만 하는게 <b>"가독성"</b> 측면에서 매우 중요합니다. 

R.id.ib_thubmb_up:<br>
   &nbsp;&nbsp;&nbsp;&nbsp;<b>handleLikeButton();</b><br>
   &nbsp;&nbsp;&nbsp;&nbsp;break;<br>
R.id.ib_thub_donw:<br>
   &nbsp;&nbsp;&nbsp;&nbsp;<b>handleDisLikeButton();</b><br>
   &nbsp;&nbsp;&nbsp;&nbsp;break;
> 위와 같이 <b>함수를 추출</b>하여 코드를 작성하면 더 읽기 쉬운 코드가 됩니다. <b>함수는 최대한 짧게, 함수명은 주석 없이도 읽기 쉽게 하는 것</b>을 유념하고 작성하세요.
