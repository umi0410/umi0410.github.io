---
title: "Go의 Pipeline pattern. 언제 사용해야할까? - Golang concurrent patterns"
menuTitle: Go의 Pipeline pattern. 언제 사용해야할까?
date: 2020-01-11T12:46:54+09:00
chapter: false
pre: "<b></b>"
weight: 20
noDisqus: false
draft: false
description: |
  Golang의 Concurrent pattern 중 하나로서 Channel을 바탕으로 커뮤니케이션하며 작업을 진행하는 패턴인
  Pipeline pattern에 대해 간략해 소개해봤고, 언제 사용해야할지 그 use case에 대해 알아보았습니다. 
---
{{% toc %}}

## ✋ 시작하며

`Go`를 공부하기 시작한 지도 벌써 몇 달이 지난 것 같다. 데브옵스 인턴을 마치면서 특히나 관심있었던 Go를 공부하기 시작했었고, 지난 몇 달간 AWS KRUG내의 소모임인 AUSG의 스터디 활동으로 Go를 주제로 공부해왔다. Golang의 꽃이라고 할 수 있는 요소들이 몇 개 있었는데 나는 그 중 `goroutine`과 `channel`에서 매력을 느꼈고 그를 바탕으로한 `concurrency pattern`들에 대해 이래 저래 많이 알아봐왔다.

하지만 concurrency pattern이 뭐고, 어떻게 사용하는지에 대해서는 여러 글을 찾아볼 수 있었지만 이게 왜 좋고 언제 쓰면 좋을지에 대한 내용은 찾아보기 힘들었다. 항상 어떤 기술을 접할 때 "*왜 좋은데?*"와 "*언제 쓰면 좋은데?*"를 많이 따지는 편이라서 늘 궁금증에 남아있었다.

그러던 중 얼마 전 나의 궁금증을 해소시켜주는 간단한 댓글을 보게 되었고, 그를 바탕으로 몇 가지 서치를 해본 결과 concurrency pattern 중 하나인 `pipeline pattern`을 언제 쓰면 좋을지 알아보았다.

## ❓ Pipeline pattern이란?

![illustration.png](illustration.png)

Pipeline pattern은 golang의 Concurrency pattern 중 하나이다. 쉽게 말하자면 함수의 인자에 값을 전달해 작업하는 것이 아니라 `channel` 을 통해 실시간으로 **함수(혹은 goroutine)들끼리 커뮤니케이션을 하며 작업을 진행**하는 것이다. 실시간으로 함수들끼리 커뮤니케이션하며 data를 전송하기 때문에 data streaming 같은 느낌이라고 볼 수 있겠다. 

일반적으로는 한 함수에서 작업이 모두 끝난 뒤 그 결과값을 리턴하고, 또 다시 그 결과 값을 인풋으로 어떤 함수가 작업을 하고 해당 작업이 모두 끝난 뒤 또 다른 결과값을 리턴하는 형태로 진행이 된다. 하지만 Pipeline pattern에서는 한 작업이 모두 완료되지 않았다하더라도 해당 작업의 부분 부분의 데이터를 담는 channel을 리턴하고, 다른 작업은 해당 channel을 인풋으로 하여 부분 부분의 데이터를 channel에서 꺼내 바로 바로 작업한다.

![python-generator.png](python-generator.png)

*<center>generator을 이용한 lazy evaluation. 0, 1, 2를 모두 Insert한 뒤 square and print 하는 것이 아닌 하나씩 실시간으로 진행</center>*

이는 마치 **Python의 `Generator`을 통한 `Lazy evaluation`을 이용할 때와 유사**하게 볼 수는 있겠다. 하지만 많은 차이점이 존재하긴 할텐데 우선 generator는 thread-safe하지 않은([stackoverflow 참고](https://stackoverflow.com/questions/1131430/are-generators-threadsafe)) 반면 channel은 여러 goroutine에서 한 channel에 접근해도 thread-safe하다. 'thread-safe한가'가 중요한 이유는 후에 잠깐 소개할 `fan-in fan-out pattern`이 바로 pipeline pattern이 여러 goroutine에서 이루어지는 경우이기 때문이다.

(*사실 내가 Pipeline pattern을 어디에 사용할 지 몰랐던 이유는 Lazy evaluation과 같은 개념이 없었기 때문이 아닐까싶기도 하다.*)

### Pipeline pattern의 흔한 예시로 square 하는 예제를 설명하는 글들

종종 Pipeline pattern의 예시를 알아보면서 square 작업을 하는 예제들을 몇 개 봤던 것 같은데, 그 중 참고할 수 있는 예시들을 제시해본다. square 작업이 pipeline pattern에 효율적이라기보다는 그냥 임의의 예시라고 생각한다.

이 글은 Pipeline pattern을 언제 사용하면 좋을지에 초점을 맞추었기에 Pipeline pattern이 뭔지, 그 사용법은 어떻게 되는지 등의 내용을 모르는 상태라면 아래 글들을 추천한다. (특히 Aidan Bae님의 블로그에는 pipeline pattern 외에도 다양한 Go 관련 내용을 잘 적혀있다.)

- Aidan Bae님의 아주 간단한 Pipeline pattern 예시 - [https://aidanbae.github.io/code/golang-design/pipeline/](https://aidanbae.github.io/code/golang-design/pipeline/)
- GoBlog의 간단한 내용부터 꽤나 어려운 내용까지 들어가는 것 같은데, 내용도 길고  짧은 코드에 억지로 많은 내용을 끼워넣은 느낌.. [https://blog.golang.org/pipelines](https://blog.golang.org/pipelines)

## 🤔 언제 쓰는 게 좋을까?

앞선 lazy evaluation 관련된 내용과 [이 글의 댓글](https://www.reddit.com/r/golang/comments/7rjdw6/go_go_go_stream_processing_for_go/)(go로 작업을 stream하라는 글에 "*너 정말 저렇게 stream하는 게 좋다고 생각해? 이 경우 아니면 그닥 잘 모르겠는데?*"라는 댓글)이 나의 이해를 도와주었다. [Pipeline architecture를 사용해야하는 7가지 이유라는 글](https://labs.bawi.io/7-reasons-to-use-pipeline-architecture-93346f604b87)을 읽어보기도 했지만 여기의 내용은 실질적으로 왜 그 장점을 갖는지에 대한 설명이나 논리가 부족했다. (*~~다소 Go와 Pipeline pattern에 대한 무조건적 사랑으로 느껴짐..ㅎㅎ...~~*)

내 생각에는 아래 두 가지 경우에 Pipeline pattern을 이용하는 것이 좋을 것 같다.

- **Input 작업을 모두 수행하는 데에 오랜 시간이 걸리는 경우**
- **작업하는 데이터의 양이 너무 커서 Memory를 많이 점유하므로 쪼개서 바로 바로 처리하고 싶은 경우**

'*디버깅하기 쉽다*'거나 '*코드를 알아보기 쉽다*' 등등의 글을 보긴했던 것 같은데 개인적으로는 그냥 pipeline과 channel을 안 쓰고 그냥 반복문을 돌리면서 데이터를 통째로 작업하는 게 디버깅이나 가독성면으로 훨씬 유리하다고 생각한다. 즉 Concurrent pattern의 장점은 특정 use case에서의 성능적인 측면이라고 생각한다.

그리고 이건 여담인데 **go의 channel을 통한 data 전달이 일반적으로 빠른 건 절대 아니다.** channel은 우선 `thread safe`하기때문에 일반적인 작업에서는 **thread-unsafe한 방식보다 느린 것이 당연**하다. 또한 channel은 단순한 lock 기능 외에도 다른 goroutine(혹은 thread 혹은 function) 간의 데이터 전송, channel에 대한 반복, 열었는지 닫았는지에 대한 체크 등등 다양한 기능도 갖고있기때문에 **mutex에 비해서도 훨씬 느린 듯하다**. 그렇기 때문에 위에서 나열했듯 Input 작업이 오래 걸리거나 Memory 점유량을 줄이기 위한 경우에는 channel을 이용한 Pipeline pattern으로 실시간으로 작업하면 유리할 것이라고 생각한 것이다.

그리고 추가적으로 앞에서 `Fan-in Fan-out` pattern에서는 여러 Goroutine에 대한 Pipeline pattern이 적용된다고했는데, Fan-in Fan-out이란 여러 goroutine이 하나의 channel에 값을 넣거나 빼는 구조를 말한다. 이 구조의 장점은 일반적인 Pipeline Pattern에서 각 단계가 하나의 goroutine만을 이용하는 것이 아니라 여러 goroutine을 이용할 수 있다는 점이다. CPU bound한 작업이 아닌 IO Block이 주요 latency를 차지하는 경우는 Goroutine을 늘려주면 concurrent하게 작업할 수 있기때문에 성능이 좋아지더라. Fan-in Fan-out pattern에 대해서는 기회가 된다면 더 자세히 다뤄보겠다.

**참고**

- Mutex보다 channel이 느리다는 벤치마크 글 - [http://www.dogfootlife.com/archives/452](http://www.dogfootlife.com/archives/452)
- Mutex를 쓸 수 있는 상황이면 Mutex를 쓰는 것이 좋을 수도 있다 - [https://github.com/golang/go/wiki/MutexOrChannel](https://github.com/golang/go/wiki/MutexOrChannel)

## 예시 프로그램

### 프로그램 설명

Input 작업을 수행하는 데에 오랜 시간이 걸리는 경우를 예시로 들기 위해 MySQL에서 Gopher에 대한 데이터를 조회한 뒤 Gopher들의 연령을 +10 증가시키는 예시를 만들어봤다. 

Input 작업을 모두 수행하는 데에 오래 걸리고, 작업 내용을 조금씩 쪼갤 수 있다는 전제조건을 위해 DB Query를 할 때 전체를 한 번에 Select 하는 것이 아니라 1개씩 Select하는 다소 비효율적이고 비현실적인 상황을 이용하긴하지만 간단하게 Pipeline pattern의 효율을 극대화시켜보고자했던 이유이므로 양해를 부탁드린다. 

예를 들어 Pipeline pattern을 통해 **1000개의 Row를 Query하고 난 뒤 1000개를 Update하는 것이 아니라 1000개의 Row를 Query하면서 1개 1개의 Row를 얻어올 때 마다 바로바로 Update 작업에 Row를 넘겨줘 실시간으로 작업**할 수 있게해주는 것이다.

- Pipeline pattern: 한 item 씩 실시간으로 전달하며 진행
- Sequential pattern: 딱히 Pattern이라기엔 좀 그렇지만 그냥 일반적으로 Sequential하게 진행하는 경우를 의미. 이 예시에선 전체 Query 완료 후 전체 Update하는 방식.

Docker를 이용해 간단하게 MySQL 서버를 띄웠고, Gorm을 이용해 DB Query와 update를 수행했다.

{{%expand "예시 프로그램 코드" %}}

```go
package main

import (
	"fmt"
	"github.com/brianvoe/gofakeit/v6"
	"gorm.io/driver/mysql"
	//"gorm.io/driver/sqlite"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
	"math/rand"
	"time"
)

var (
	NumTotalData      int = 100 // 초기화할 데이터
	NumManipulateData int = 100 // fetch 후 map을 적용할 데이터 수
)

type Gopher struct {
	gorm.Model
	Name string
	Age  int
}

func initializeDB(db *gorm.DB) {
	// Migrate the schema
	db.AutoMigrate(&Gopher{})

	// Create initial data
	for i := 0; i < NumTotalData; i++ {
		gopher := &Gopher{Name: gofakeit.Name(), Age: rand.Intn(60) + 1}
		gopher.ID = uint(i + 1)
		db.Save(gopher)
	}
	fmt.Println("Initialized DB")
}

type SequentialPattern struct {
	DB *gorm.DB
}
type PipelinePattern struct {
	DB *gorm.DB
}

func (sp *SequentialPattern) Execute() {
	start := time.Now()
	sp.Map(sp.FetchData())
	fmt.Println("sequential", sp)
	fmt.Println("Elapsed:", time.Now().Sub(start))
	fmt.Print(time.Now().Sub(start))
	gophers := make([]Gopher, 0)
	sp.DB.Limit(NumTotalData).Find(&gophers)
	fmt.Printf("Average age(Num: %d): %d\n", len(gophers), func() int{
	  totalAge := 0
	  for _, gopher := range gophers{
	      totalAge += gopher.Age
	  }
	  return totalAge/len(gophers)
	}())
}

func (sp *SequentialPattern) FetchData() []*Gopher {
	gophers := make([]*Gopher, NumManipulateData)
	for id := range gophers {
		gophers[id] = &Gopher{}
		sp.DB.Where("id > ?", id).Limit(1).Find(&gophers[id])
	}

	return gophers
}

func (sp *SequentialPattern) Map(gophers []*Gopher) {
	for _, gopher := range gophers {
		gopher.Age += 10
		sp.DB.Save(gopher)
	}
}

func (pp *PipelinePattern) Execute() {
	start := time.Now()

	pp.Map(pp.FetchData())
	fmt.Println("pipeline", pp)
	fmt.Println("Elapsed:", time.Now().Sub(start))
	gophers := make([]Gopher, 0)
	pp.DB.Limit(NumTotalData).Find(&gophers)
	fmt.Printf("Average age(Num: %d): %d\n", len(gophers), func() int{
	   totalAge := 0
	   for _, gopher := range gophers{
	       totalAge += gopher.Age
	   }
	   return totalAge/len(gophers)
	}())
}
func (pp *PipelinePattern) FetchData() <-chan *Gopher {
	ch := make(chan *Gopher)
	go func() {
		for id := 0; id < NumManipulateData; id++ {
			var gopher Gopher
			pp.DB.Table("gophers").Where("id > ?", id).Limit(1).Find(&gopher)
			ch <- &gopher
		}
		close(ch)
	}()

	return ch
}

// 고퍼들을 모두 10살 더 먹게 함.
func (pp *PipelinePattern) Map(gophers <-chan *Gopher) {
	for gopher := range gophers {
		gopher.Age += 10
		pp.DB.Save(gopher)
	}
}

func main() {
	// $ docker run -it --name mysql --rm -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=concurrency -p 3306:3306 mysql
	// 로 mysql 서버를 띄울 수 있다.
	db, err := gorm.Open(mysql.Open("root:root@tcp(127.0.0.1:3306)/concurrency?charset=utf8mb4&parseTime=True&loc=Local"),
		&gorm.Config{Logger: logger.Default.LogMode(logger.Silent)})
	if err != nil {
		fmt.Println(err)
		panic("failed to connect database")
	}
	initializeDB(db)

	// Execute with PipelinePattern
	func() {
    fmt.Println("==============================")
		for rep := 0; rep < 10; rep++ {
			pattern := &SequentialPattern{DB: db}
			pattern.Execute()
		}
    fmt.Println("==============================")
	}()

	fmt.Print("DB 부하를 가라앉히기 위해 5초간 휴식")
	time.Sleep(5 * time.Second)
	// Execute with PipelinePattern
	func() {
    fmt.Println("==============================")
		for rep := 0; rep < 10; rep++ {
			pattern := &PipelinePattern{DB: db}
			pattern.Execute()
		}
    fmt.Println("==============================")
	}()
}
```

{{% /expand %}}

### 📉 벤치마크 결과 비교

![line-chart-1.png](line-chart-1.png)

Sequential하게 진행할 경우 모든 Item에 대한 Query가 완료될 때 까지 Update 작업은 **지연**되게 된다. 그리고 Query가 모두 완료되어야 비로소 Update 작업을 시작할 수 있게되므로  좀 더 작업 시간이 오래걸리는 편이다.

반면 **Pipeline**으로 진행할 경우 한 Item을 Query 하자마자 **바로 바로** Upadte 작업이 이루어질 수 있기때문에 작업 시간이 더 짧은 경향이 있다.

하지만 이 정도 차이는 뚜렷한 성능 차이로 보기엔 다소 미미한 것 같았다. 그래서 좀 더 IO latency가 긴 상황을 이용해봤다. DB로 localhost에서 docker mysql server를 이용하는 것이 아니라 운영 중이던 k8s cluster의 mysql pod를 kubectl port forward하여 이용해보았다. (*정확히는 모르지만 remote db + kubectl port forawrd 를 이용하는 경우 network latency가 아주 커져 극단적으로 좋은 예시 상황이 될 수 있을 것 같았다.*)

![line-chart-2.png](line-chart-2.png)

위에서 가정한 remote db + port forward의 극단적 상황은 작업 진행 pattern에 따라 latency를 2배 가량 차이나게 했다.

즉 **한 단계에서 오랜 시간이 소모되어 다음 단계가 지연되는 경우 Pipeline을 이용하면 좋은 것 같다**. *이는 마치 컴퓨터 구조에서 MIPS Processor의 Pipeline을 공부할 때와 유사한 듯한 느낌을 줬다.*

## 마무리

> 이 글이 정확한 내용은 아닐 수 있지만 조사해본 선에서 벤치마크 해보며 작성해봤습니다. 혹시 글의 내용 중 잘못된 내용에 대한 피드백을 제시해주신다면 기회가 닿는 한 열심히 다시 알아보겠습니다~!

평소 궁금했던 "*Go의 `concurrent pattern`은 어떨 때 쓰면 좋을까?*" 라는 의문 중 `pipeline pattern` 에 대해 이렇게 알아봤다. 

Pipeline pattern이 CPU Bound한 작업보다는 IO latency로 인해 오랜 시간이 소모되는 작업에 더 효율적이라고 생각한 이유는 CPU Bound한 작업에서는 CPU가 이미 혹사당하고 있기 때문에 굳이 여러 goroutine을 schedule, switch하거나 channel을 통한 동기처리를 하면서 작업을 진행하기보다는 그냥 일반적인 방식으로 순차적으로 작업을 진행하는 게 좋을 수 있기 때문이다. 워낙에 얼마나 CPU bound한 작업인지, 얼마나 많은 goroutine을 이용하는지 등등 다양한 경우에 따라 달라지기 때문에 딱 잘라말할 수도 없고, 나도 많이 부족하기 때문에 정확히는 모르겠다. 하지만 추측컨대 적어도 io latency로 인해 작업들이 지연되는 경우에는 pipeline pattern이 좋은 듯하다.

추가적으로 `Fan-in Fan-out`  pattern 또한 channel을 바탕으로 data를 전달하면서 작업하기 때문에 일종의 `Pipeline` pattern이 적용된 패턴인듯하여 기회가 된다면 **어떤 경우에 단순히 Pipeline 패턴을 이용하는 것보다 여러 Goroutine을 이용해 작업하는 Fan-in fan out pattern이 좋을지 비교**해보는 글을 적어보고싶다.

## 참고

- Aidan Bae님의 Pipeline pattern에 대한 소개 - [https://aidanbae.github.io/code/golang-design/pipeline/](https://aidanbae.github.io/code/golang-design/pipeline/)
- Go Blog의 Pipeline pattern에 대한 소개 - [https://blog.golang.org/pipelines](https://blog.golang.org/pipelines)
- Pipeline은 ~~~ 이런 상황에 도움이 될 껄?하는 댓글 - [https://www.reddit.com/r/golang/comments/7rjdw6/go_go_go_stream_processing_for_go/](https://www.reddit.com/r/golang/comments/7rjdw6/go_go_go_stream_processing_for_go/)
- Mutex와 Channel 비교 - [https://github.com/golang/go/wiki/MutexOrChannel](https://github.com/golang/go/wiki/MutexOrChannel)
- Mutex와 Channel의 속도 차이 - [http://www.dogfootlife.com/archives/452](http://www.dogfootlife.com/archives/452)
- Pipeline architecture를 사용하는 7가지 이유(~~개인적으론 공감 안 감~~) - [https://labs.bawi.io/7-reasons-to-use-pipeline-architecture-93346f604b87](https://labs.bawi.io/7-reasons-to-use-pipeline-architecture-93346f604b87)
- python generator 소개, 사용법- [https://wikidocs.net/16069](https://wikidocs.net/16069)
- python generator는 thread-safe하지 않다. - [https://stackoverflow.com/questions/1131430/are-generators-threadsafe](https://stackoverflow.com/questions/1131430/are-generators-threadsafe)
