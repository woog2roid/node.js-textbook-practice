# 프론트 엔드
크게 중요하게는 여기지 않아도 됨...
- __./public:__ main.css 들어있음
- __./veiws:__ .html 파일들이 들어있다.
- __./router/page.js:__ page.js로 html 파일들을 라우팅 해준다.

# 데이터베이스 세팅
- 자동으로 ./models/index.js와 config/config.json이 생성됨
- config.json에는 설정에 관한 정보들이 있음
- __index.js에서는 db라는 객체에 모델을 담아두어서 db 객체를 require해서 모델들에 접근이 가능해지게 한다.__

__모델은 user, post, hashtag, postHashtag, follow로 이루어져 있음__
- user: email, nickname, password, provider, snsID로 이루어짐
- post: content, img로 이루어짐
- hashtag: title로 이루어짐
- 시퀄라이즈에서 생성한 PostHashtag, Follow도 모델로 생성된다.

__관계__
1. user <-> post => 1:N
2. post <-> hashtag => N:M
3. user <-> user (팔로우) => N:M
    - as옵션을 통해 구분해주었음.

__데이터 베이스 생성:npx sequelize db:create__

# Passport 모듈로 Local로그인 & 카카오 로그인 구현
- npm i passport passport-local passport-kakao bcrypt으로 passport 설치

__index.js 분석__
- serialize는 로그인시에만 이용됨
  - req.sesssion 객체에 어떤 데이터를 저장할지 정하는 메서드임
- deserialize는 매 요청시마다 실행됨
  - serialize에서 세션에 저장했던 아이디를 받아 DB에서 사용자 정보를 조회하고 done에서 조회한 정보를 req.user에 저장한다.

```javascript
module.exports = () => {
    passport.serializeUser((user, done) => {
        done(null, user.id);//두번 째 인수인 id가
    });
    passport.deserializeUser((id, done) => {//여기의 첫번째 인수로 들어간다.
        User.findeOne({where : {id}})
            .then(user => done(null, user))
            .catch(err => done(err))
    })
}
```
- sns의 업로드 기능 등은 로그인 한 유저만 이용해야 하기 때문에 middlewares.js에 이를 구분하는 기능을 만들었다.
- localStrategy에서는 new LocalStrategy에서 첫번째 인수로는 객체를, 두번째 인수로는 전략을 수행하는 async 함수를 넣어주면 된다.