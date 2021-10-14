# 🚀 조회 성능 개선하기

## A. 쿼리 연습

### * 실습환경 세팅

```sh
$ docker run -d -p 23306:3306 brainbackdoor/data-tuning:0.0.1
```
- [workbench](https://www.mysql.com/products/workbench/)를 설치한 후 localhost:23306 (ID : user, PW : password) 로 접속합니다.

<div style="line-height:1em"><br style="clear:both" ></div>

> 활동중인(Active) 부서의 현재 부서관리자 중 연봉 상위 5위안에 드는 사람들이 최근에 각 지역별로 언제 퇴실했는지 조회해보세요.
(사원번호, 이름, 연봉, 직급명, 지역, 입출입구분, 입출입시간)

1. 쿼리 작성만으로 1s 이하로 반환한다.
2. 인덱스 설정을 추가하여 50 ms 이하로 반환한다.

### 쿼리작성(인덱스 적용 X)
```sql
select 
    사원.사원번호 as 사원번호, 
    사원.이름 as 이름,
    상위연봉부서관리자.연봉 as 연봉,
    직급.직급명 as 직급명,
    사원출입기록.입출입시간 as 입출입시간,
    사원출입기록.지역 as 지역,
    사원출입기록.입출입구분 as 입출입구분
from 
    사원 
inner join 
	(select
	    부서관리자.사원번호,
    	급여.연봉
	from 
        부서
	inner join 
        부서관리자 on 부서.부서번호 = 부서관리자.부서번호
	inner join 
        급여 on 급여.사원번호 = 부서관리자.사원번호
	inner join 
        사원 on 사원.사원번호 = 부서관리자.사원번호
	where 
        부서.비고 = 'active'
    and 
        급여.종료일자 > now()
    and 
        부서관리자.종료일자 > now()
	group by 부서관리자.사원번호
	order by 급여.연봉 desc
	limit 5) as 상위연봉부서관리자
on 상위연봉부서관리자.사원번호 = 사원.사원번호

inner join 
    사원출입기록 on 사원.사원번호 = 사원출입기록.사원번호 and 사원출입기록.입출입구분 = 'O'
inner join 
    직급 on 사원.사원번호 = 직급.사원번호 and 직급.직급명 = 'Manager'
order by 연봉 desc;
```
### 현재 결과
<img width="154" alt="스크린샷 2021-10-14 오전 10 15 10" src="https://user-images.githubusercontent.com/56679885/137234103-cf1acddc-4965-4630-9a20-cde47b4fc9b0.png">

Duration: 0.508 ~ 0.530 sec

#### 실행계획
**시각화**
<img width="1397" alt="스크린샷 2021-10-13 오후 9 01 20" src="https://user-images.githubusercontent.com/56679885/137128563-ecdd266f-250f-4af7-846a-535552cfa142.png">

**Explain**
<img width="1355" alt="스크린샷 2021-10-15 오전 1 32 55" src="https://user-images.githubusercontent.com/56679885/137359398-930d62c5-6278-4cb6-9880-7049e5d4454f.png">

### 인덱스 적용
`사원출입기록`을 join 하는 과정에서  

```sql
ALTER TABLE `tuning`.`사원출입기록` 
ADD INDEX `I_사원번호_입출입기록` (`사원번호` ASC, `입출입구분` ASC);
```

#### 실행계획

### 현재 결과


<div style="line-height:1em"><br style="clear:both" ></div>
<div style="line-height:1em"><br style="clear:both" ></div>

## B. 인덱스 설계

### * 실습환경 세팅

```sh
$ docker run -d -p 13306:3306 brainbackdoor/data-subway:0.0.2
```
- [workbench](https://www.mysql.com/products/workbench/)를 설치한 후 localhost:13306 (ID : root, PW : masterpw) 로 접속합니다.

<div style="line-height:1em"><br style="clear:both" ></div>

### * 요구사항

- [ ] 주어진 데이터셋을 활용하여 아래 조회 결과를 100ms 이하로 반환

    - [ ] [Coding as a  Hobby](https://insights.stackoverflow.com/survey/2018#developer-profile-_-coding-as-a-hobby) 와 같은 결과를 반환하세요.

    - [ ] 각 프로그래머별로 해당하는 병원 이름을 반환하세요.  (covid.id, hospital.name)

    - [ ] 프로그래밍이 취미인 학생 혹은 주니어(0-2년)들이 다닌 병원 이름을 반환하고 user.id 기준으로 정렬하세요. (covid.id, hospital.name, user.Hobby, user.DevType, user.YearsCoding)

    - [ ] 서울대병원에 다닌 20대 India 환자들을 병원에 머문 기간별로 집계하세요. (covid.Stay)

    - [ ] 서울대병원에 다닌 30대 환자들을 운동 횟수별로 집계하세요. (user.Exercise)

<div style="line-height:1em"><br style="clear:both" ></div>
<div style="line-height:1em"><br style="clear:both" ></div>

## C. 프로젝트 요구사항

### a. 페이징 쿼리를 적용 

### b. Replication 적용 
