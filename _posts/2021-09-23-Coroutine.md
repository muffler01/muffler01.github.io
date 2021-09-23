---
title: "Coroutine"
excerpt: "Coroutine"

categories:
  - Unity
tags:
  - Unity:Programming

header:
  teaser: /assets/images/etcImage/muffler.jpg

toc: true
toc_sticky: true
toc_label: "Contents"


last_modified_at: 2021-09-23T09:30
---

# Unity, Coroutine

Unity에서 딜레이를 주고자 할때, 주로 사용되는 Coroutine의 사용법에 대해 정리하였다.  
문법을 알고 있는 것도 중요하지만, 더욱 중요한 것은 활용 방법에 대해 생각해보자.

### Coroutine의 사용 이유

- Unity에서 가장 기본적인 동작 처리는 일반적으로 Update() 함수에서 처리한다.  
  하지만 Update() 함수는 정지 없이 매번 호출되는 함수이기 때문에 딜레이를 주거나 서브 동작을 처리하기 어렵다.  
  또한, Update() 함수에 모든 서브 동작을 처리하게 된다면 가독성이 떨어지며 관리하기 힘들어진다.
- 따라서, 서브 동작 처리나 딜레이를 주고자 할 때에는 Coroutine을 사용하여 처리한다.

### Coroutine을 사용한 예시: 공격 딜레이

- Update() 함수를 이용한 공격 딜레이

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Attacker : MonoBehaviour
{
    public bool isDelay;	// 딜레이인지 확인하기 위한 bool 값
    public float delayTime = 2f;	// 딜레이 시간
    
    float timer = 0f; // 딜레이 시간 체크를 위한 변수
    
    void Start()
    {
        
    }    

    void Update()
    {
        if(Input.GetKeyDown(KeyCode.Space))
        {
            if(!isDelay)	// 딜레이가 아니라면, 딜레이를 걸어두고 공격을 실행 
            {
                isDelay = true;
                Debug.Log("Attack");
            }
            else	// 딜레이라면, 딜레이 실행
            {
                Debug.Log("Delay");
            }
        }
        
        if(isDelay)	// 딜레이라면, 타이머에 계속해서 시간을 더해준다.
        {
            timer += Time.deltaTime;
            if(timer >= delayTime) // 기존 딜레이(2초)보다 크거나 같아지면 타이머를 초기화하고 딜레이를 비활성화한다.
            {
                timer = 0f;
                isDelay = false;
            }
        }
    }
}
```

- Coroutine을 이용한 공격 딜레이

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Attacker : MonoBehaviour
{
    public bool isDelay;	
    public float delayTime = 2f;	
    
    float timer = 0f; 
    
    void Start()
    {
        
    }
    
    void Update()
    {
        if(Input.GetKeyDown(KeyCode.Space))
        {
            if(!isDelay)	
            {
                isDelay = true;
                Debug.Log("Attack");
               	StartCoroutine(CountAttackDelay());	// 코루틴 호출
            }
            else	
            {
                Debug.Log("Delay");
            }
        }
    }
    
    // 기본적인 Coroutine 사용방법
    IEnumerator CountAttackDelay()
    {
        yield return new WaitForSeconds(delayTime);
        isDelay = false;
    }
}
```

- Coroutine은 반환형을 IEnumerator로 취한다. 
- yield return으로 특정 시간동안만 Unity에 관리를 맡길 수 있다. 쉽게 말해 딜레이 시간을 조정 할 수 있다.  
  null의 경우는 1프레임, WaitForSeconds는 초 단위의 시간을 설정할 수 있다.
- WaitForSeconds는 기본형과 Realtime형이 있는데 Realtime형은 TimeScale의 영향을 받지 않는다.
- Coroutine를 호출하기 위해서는 StartCoroutine()을 이용해야 한다.
- StartCoroutine()을 사용할 때, 함수의 이름만으로도 실행가능하다.

```cs
	StartCoroutine("CountAttackDelay");
```

### Coroutine 사용 시 주의사항

- Start() 함수 호출 전인 Awake() 함수에서 호출할 때는 작동하지 않는다.
- 게임 오브젝트가 비활성화된 상태에는 작동하지 않는다.
- Coroutine 내에서 무한루프를 돌릴 때, 반복문 밖에 yield return null; 을 사용하면 게임이 멈춘다.

```cs
    IEnumerator InfinityLoop()
    {
		while(true)
        {
            yield return null; // 올바른 사용법:내부에 구문을 적어 제어권을 양보해야한다.
        }
    }
```

### Coroutine 멈추기

- StopCoroutine()을 이용하는 방식

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CoroutineTest : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(TestCoroutine());
    }
    
    void Update()
    {
		if(Input.GetKeyDown(KeyCode.Space))	// 스페이스바를 눌렀을 때, 코루틴을 중지
        {
            StopCoroutine(TestCoroutine);
        }
    }
    
    IEnumerator TestCoroutine()
    {
		int i = 0;
        while(true)
        {
            yield return null;
            Debug.Log("Coroutine " + i);
        	i++;
		}
    }
}
```

- 위의 코드대로 작성한다면, 스페이스바를 눌렀을 때에도 코루틴이 중지하지 않는다.
- 코루틴은 IEnumerator를 반환형으로 가지는데 Start와 Stop에서 받는 반환값이 동일하지 않기 때문에 멈추지 않는다.
- 그렇기 때문에 따로 Start에서 반환하는 IEnumerator를 변수로 저장하여 이를 Stop에 넣어주어야 한다.

```cs
	IEnumerator enumerator;

	...
        
	enumerator = TestCoroutine();
	StartCoroutine(TestCoroutine());

	...
        
    StopCoroutine(enumerator);
```

- 코루틴을 실행할 때와 마찬가지로 중지할 때에도 함수의 이름만으로 중지가 가능하다.

```cs
	StopCoroutine("TestCoroutine");
```

- 다만, 같은 이름의 코루틴이 2개 실행된다면 2개가 동시에 중단하기 때문에 상황에 따라 다르게 설계되어야 한다.
- StopAllCoroutine()을 이용하면 모든 코루틴을 중지 시킬 수 있다.

### Coroutine 매개변수

- 매개변수를 활용하는 법

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CoroutineTest : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(TestCoroutine(10)); // 매개변수 10
    }
    
    void Update()
    {
		if(Input.GetKeyDown(KeyCode.Space))	
        {
            StopAllCoroutine(TestCoroutine);
        }
    }
    
    IEnumerator TestCoroutine(int count)	// 매개변수를 받는다.
    {
		int i = 0;
        while(i < count)	// 매개변수만큼 반복
        {
            yield return null;
            Debug.Log("Coroutine " + i);
        	i++;
		}
    }
}
```

- 코루틴 함수에 받아줄 매개변수의 타입과 변수명을 선언한다.
- 함수를 호출하는 경우에는 () 뒤에 매개변수를 적어주면 된다. 
- 함수의 이름으로 호출하는 경우에는 ','을 취하고 매개변수를 적어주면 된다.

```cs
	StartCoroutine("TestCoroutine", 10);
```

- 이때 10은 object 타입인데, 이후 코루틴 함수에서 int 타입으로 변형되어 받아진다.
- 다만, object 타입을 사용하면, boxing, unboxing 과정을 동반하기 때문에 최적화에 어려움이 있을 수 있다.
- 함수의 이름으로 호출하는 경우에는 2개의 매개변수를 전달할 수 없다. 따라서 2개의 매개변수를 전달할 때에는 함수 자체를 호출하는 것이 권장된다.

### yield break

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CoroutineTest : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(ReturnCoroutine());
        StartCoroutine(BreakCoroutine());
    }
    
    void Update()
    {
        
    }
    // return
    IEnumerator ReturnCoroutine()
    {
        Debug.Log("Return 1");
        yield return null;
		Debug.Log("Return 2");
    }
    // break
    IEnumerator BreakCoroutine()
    {	
        Debug.Log("Break 1");
        yield break;
        Debug.Log("Break 2");
    }
}
```

- yield break가 호출되면 코루틴이 완전히 중지된다. 반면, return의 경우에는 일정 시간 제어권을 잃어버리지만 시간이 지나면 돌아오기 때문에 Return 2가 실행된다.
- 간단한 코루틴의 경우 yield break로 중지하는 방법이 가독성을 늘려준다.

### 추가 및 수정

- [**코루틴 다루기 (1) - 코루틴 기초, 유니티**](https://www.youtube.com/watch?v=ahji9F5hJJ4)

- [**코루틴 다루기 (2) - 코루틴 중단하기 + 코루틴 매개변수 + yield break , 유니티**](https://www.youtube.com/watch?v=iK7zDp5TEks)

