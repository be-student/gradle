# gradle

# Gradle 학습

``` kotlin
tasks.register("hello"){
    doFirst{
        println("doFirst")
    }
    println("do Middle")
    doLast {
        println("doLast")
    }
}
```
![initial](single/resource/hellotask.PNG)

출력 순서가 doMiddle->doFirst->doLast 순서로 출력된다.

명시적으로 순서를 지정하지 않으면 이해하기 어려울 수 있으니
웬만하면 doFirst, doLast 를 사용하는 것이 좋다

조금 구체적으로는 명시적으로 지정하지 않으면 configuration 단계에서
지정한다면 doFirst, doLast 를 사용하면 execution 단계에서 지정한다.

## 여러개 task를 한번에 실행하기

settings.gradle.kts
```kotlin
println("This is executed during the initialization phase.")
```
build.gradle.kts
``` kotlin
println("This is executed during the configuration phase.")

tasks.register("configured") {
    println("This is also executed during the configuration phase, because :configured is used in the build.")
}

tasks.register("test1") {
    doLast {
        println("This is executed during the execution phase.")
    }
}

tasks.register("testBoth") {
    doFirst {
        println("This is executed first during the execution phase.")
    }
    doLast {
        println("This is executed last during the execution phase.")
    }
    println("This is executed during the configuration phase as well, because :testBoth is used in the build.")
}
```
![tasks](single/resource/tasks.PNG)

## task 루트에서 특정 조건에 따라실행하기
```kotlin
allprojects {
    // Set a default value
    extra["hasTests"] = false

    afterEvaluate {
        if (extra["hasTests"] as Boolean) {
            println("Adding test task to $project")
            tasks.register("test1") {
                doLast {
                    println("Running tests for $project")
                }
            }
        }
    }
}
```
이거를 루트 경로에 build.gradle.kts 에 추가하면
gradle -q test1 을 통해 실행할 수 있다
모든 것들은 다 false 이기에 처음에는 실행되지 않지만

각각 멀티모듈에 안에 있는 build.gradle.kts 에서
```kotlin
extra["hasTests"] = true
```
를 설정할 경우 설정한 부분에 한에서 실행이 가능하다

```kotlin
gradle.afterProject {
    if (state.failure != null) {
        println("Evaluation of $project FAILED")
    } else {
        println("Evaluation of $project succeeded")
    }
}

```
를 루트에서 실행시킬 경우, 모든 테스트마다 다 실행됨.
앞쪽의 hasTests 랑 무관하게 작동함

```kotlin
tasks.whenTaskAdded {
    extra["srcDir"] = "src/main/java"
}

val a by tasks.registering

println("source dir is ${a.get().extra["srcDir"]}")
```
같은 것을 통해서 커스텀 listener를 추가할 수 있음

```kotlin
tasks.register("ok")

tasks.register("broken") {
    dependsOn("ok")
    doLast {
        throw RuntimeException("broken")
    }
}

gradle.taskGraph.beforeTask {
    println("executing $this ...")
}

gradle.taskGraph.afterTask {
    if (state.failure != null) {
        println("FAILED")
    } else {
        println("done")
    }
}
```
이렇게 됐을 때
잘못된 경우 로그는 남지는 않았지만, 실행이 되지 않는 것을 확인 가능했다
ok까지는 실행이 되고, 아닐 경우 실행이 되지 않은 것까지 볼 수 있었다