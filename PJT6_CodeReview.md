# 부스트코스 Android 학습 내용
### ◆ 인터넷 연결상태 확인하기
1) AndroidManifest.xml에 <b>ACCESS_NETWORK_STATE 권한 추가</b>
```
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```
2) 시스템 서비스 객체 중 <b>ConnectivityManager</b> 객체 사용
```
ConnectivityManager manager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
```
3) ConnectivityManager 객체의 <b>getActiveNetworkInfo() 메소드 호출</b>
 ```
 NetworkInfo networkInfo = manager.getActiveNetworkInfo();
 ```
<br><br>
# PJT6. 영화정보를 단말에 저장하기 Code Review
### ◆ 개요
+ <b>인터넷 연결된 상태</b>면 서버에서 받아온 데이터 <b>데이터베이스에 저장</b>
+ <b>인터넷 연결되지 않은 상태</b>면 데이터베이스에 <b>저장된 데이터 사용</b>
<table>
  <tr>
   <td><img src="https://user-images.githubusercontent.com/25261296/63632462-528bab80-c671-11e9-921e-08c6fbbdd188.png" width="250"></td>
   <td><img src="https://user-images.githubusercontent.com/25261296/63632463-528bab80-c671-11e9-8cf6-27bf9f37081f.png" width="250"></td>
   <td><img src="https://user-images.githubusercontent.com/25261296/63632464-528bab80-c671-11e9-94ab-7361136382a3.png" width="250"></td>
  </tr>
  <tr>
   <td align="center">인터넷이 연결된 상태</td>
   <td colspan="2" align="center">인터넷이 연결되지 않은 상태</td>
  </tr>
</table>

### ◆ Advice 1
<b>INSERT OR REPLACE 구문 사용할 것!</b>
#### ◇ 수정 전
+ 데이터 추가 및 업데이트 할 때 <b>DELETE</b>로 모든 행 삭제 후 <b>INSERT</b> 함
+ 두 개의 SQL문 사용
```
public void insertMovieList(ArrayList<MovieList> movieList) { 

    String sql = "DELETE FROM " + TABLE_MOVIE_LIST; //모든 행 삭제(DELETE)
    db.execSQL(sql);

    sql = "INSERT INTO " + TABLE_MOVIE_LIST + " VALUES (?, ?, ?, ?, ?)"; //데이터 추가(INSERT)
    ...
    db.execSQL(sql, params);
}
```
#### ◇ 수정 후
+ <b>중복된 값을 가지는 행을 추가할 때는 UPDATE, 없으면 INSERT하는 INSERT OR REPLACE 구문으로 수정</b>
```
public void insertMovieList(ArrayList<MovieList> movieList) {

    //INSERT OR REPLACE 사용
    String sql = "INSERT OR REPLACE INTO " + TABLE_MOVIE_LIST + " VALUES (?, ?, ?, ?, ?)"; 
    ...
    db.execSQL(sql, params);
}
```
### ◆ Advice 2
<b>추천하기 버튼의 경우 서버 요청 후 응답이 올 때까지 해당 버튼이 동작하지 않도록 처리할 것</b>
#### ◇ 수정 후
```
... (서버요청 전)
recommendTv.setEnabled(false); //추천하기 클릭 비활성화
... (서버요청 후)
recommendTv.setEnabled(true); //추천하기 클릭 활성화
```
### ◆ Advice 3
<b>서버에서 응답 받을 때 반드시 응답코드를 확인하고 처리할 것</b><br>
#### ◇ 수정 전
```
Gson gson = new Gson();
MovieListResult result = gson.fromJson(response, MovieListResult.class);
movieList = result.result;
dbHelper.insertMovieList(movieList);
```
#### ◇ 수정 후
> 응답코드 확인하는 코드 추가
```
Gson gson = new Gson();
MovieListResult result = gson.fromJson(response, MovieListResult.class);

if (result.code == 200) { //응답코드 확인 후 처리
    movieList = result.result;
    dbHelper.insertMovieList(movieList);
}
```
### ◆ Question & Advice 4
<b>Question</b><br>
+ 서버 요청 시 아래와 같은 메세지가 가끔씩 로그에 찍히곤 합니다. Volley 라이브러리에서 response body를 close하는 방법이 있나요?
``` 
W/OkHttpClient: A connection to http://boostcourse-appapi.connect.or.kr:10000/ was leaked. 
Did you forget to close a response body? 
```
<b>Answer</b><br>
+ 해당 메시지는 Glide 라이브러리에서 OkHttp를 사용하고 OkHttp 내부에서 발생하는 메시지

+ response body를 읽고나서 해당 stream을 닫지 않아서 발생하는 메시지인데 Glide 라이브러리 내부의 OkHttp에서 발생시키므로 현재 app에서 코드 대응 처리는 불가할 것

+ Glide나 OkHttp는 이미지만 다운로드하기에는 너무 무거운 라이브러리

+ 따라서, asynctask 등을 사용하여 <b>image downloader를 직접 구현하는 방법을 권장함</b>

#### ◇ 수정 후
+ Glide 대신 <b>Volley 라이브러리의 ImageLoader 클래스를 사용</b>하여 이미지 로드
> ImageDownloader.java
```
public ImageRequest getImageRequest() {
    ImageRequest request = new ImageRequest(
            imageUrl,
            new Response.Listener<Bitmap>() {
                @Override
                public void onResponse(Bitmap response) {
                    imageView.setImageBitmap(response);
                }
            }, 0, 0, null,

            new Response.ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    Log.d("ImageDownloader", "error : " + error.getMessage());
                }
            }
    );
    request.setShouldCache(false);
    return request;
}
```
> MovieFragment.java
```
ImageDownloader downloader = new ImageDownloader(posterIv, data.getThumb());
requestQueue.add(downloader.getImageRequest());
```
+ ImageLoader를 사용하려 했으나 이미지 로딩 시간이 오래걸려 Picasso 라이브러리를 사용하기로 함
### ◆ Advice 5
MVC 패턴으로 보면 M(Model)에 대한 부분이 대부분 C(Controller)에 녹여져 있음<br>
<b>코드 변경 및 수정과 유지보수를 고려하여 Network나 Database 관련 코드들은 UI단과 별개의 모듈로 분리하여 관리할 것!!</b><br><br>
&lt;리뷰어님이 이해를 위해 그려주신 도식&gt;<br>
<img src="https://user-images.githubusercontent.com/25261296/63632798-5f5fcd80-c678-11e9-9ba1-bc5d20115700.PNG" width="650">
#### ★ 학습 중 발생한 이슈 ★
앱 실행 중 <b>OS가 안드로이드 9.0(Pie)</b>인 기기로 서버 요청을 할 때 아래와 같은 에러가 발생하였다<br>
```
java.io.IOException: Cleartext HTTP traffic to boostcourse-appapi.connect.or.kr not permitted
```
<b>안드로이드 9.0에서는 https를 사용하도록 강제</b>하는데 부스트코스에서 제공 받은 API 주소는 http이기 때문에 발생한 것이었다!<br><br>
<b>해결 방법 1 : 네트워크 주소를 https로 변경할 것<br>
=> https로 변경하여 실행시켰으나 아래와 같은 또 다른 에러가 발생 (실패) </b>
```
javax.net.ssl.SSLHandshakeException: Handshake failed
```
<b>해결 방법 2 : AndroidManifest.xml에 usesCleartextTraffic 속성을 true로 설정할 것 (성공!) </b>
```
<application
    ...         
    android:usesCleartextTraffic="true">
```
##### [참고] https://gun0912.tistory.com/80
