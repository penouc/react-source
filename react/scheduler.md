# scheduler 源码阅读

## 全局变量

|            字段            |                                         解释                                         |
| :------------------------: | :----------------------------------------------------------------------------------: |
|  enableSchedulerDebugging  |                            是否允许 Scheduler 可以 debug                             |
|     ImmediatePriority      |                           值为 1，即时优先级，最高的优先级                           |
|    UserBlockingPriority    |                                值为 2，用户阻塞优先级                                |
|       NormalPriority       |                                  值为 3，普通优先级                                  |
|        LowPriority         |                                  值为 4， 低优先级                                   |
|        IdlePriority        |                                值为 5 ，空闲时优先级                                 |
|     maxSigned31BitInt      | // Math.pow(2, 30) - 1，0b111111111111111111111111111111， 1073741823 最大无符号整型 |
| IMMEDIATE_PRIORITY_TIMEOUT |                    // Times out immediately， 值为 -1， 立即过期                     |
|   USER_BLOCKING_PRIORITY   |              // Eventually times out 值为 250 ，单位毫秒，250 毫秒过期               |
|  NORMAL_PRIORITY_TIMEOUT   |                           普通任务过期时间，5000 毫秒过期                            |
|    LOW_PRIORITY_TIMEOUT    |                           低优先级过期时间，10000 毫秒过期                           |
|       IDLE_PRIORITY        |                             永不过期时间，大概 12 天左右                             |

{% method -%}

## unstable_runWithPriority

**传入参数**

- priorityLevel 优先级
- eventHandler 事件处理程序

**方法介绍**

> 首先对传入的优先级进行判断，并且通过 switch 语句来修改全局变量 currentPriorityLevel 的值，并且执行传入的事件处理程序

{% sample lang="js" -%}

```javascript
function unstable_runWithPriority(priorityLevel, eventHandler) {
  switch (priorityLevel) {
    case ImmediatePriority:
    case UserBlockingPriority:
    case NormalPriority:
    case LowPriority:
    case IdlePriority:
      break;
    default:
      priorityLevel = NormalPriority;
  }

  var previousPriorityLevel = currentPriorityLevel;
  currentPriorityLevel = priorityLevel;

  try {
    return eventHandler();
  } finally {
    currentPriorityLevel = previousPriorityLevel;
  }
}
```

{% endmethod %}

{% method -%}

## unstable_next

**传入参数**

- eventHandler 事件处理程序

**方法介绍**

>

{% sample lang="js" -%}

```javascript
function unstable_next(eventHandler) {
  var priorityLevel;
  switch (currentPriorityLevel) {
    case ImmediatePriority:
    case UserBlockingPriority:
    case NormalPriority:
      // Shift down to normal priority
      priorityLevel = NormalPriority;
      break;
    default:
      // Anything lower than normal priority should remain at the current level.
      priorityLevel = currentPriorityLevel;
      break;
  }

  var previousPriorityLevel = currentPriorityLevel;
  currentPriorityLevel = priorityLevel;

  try {
    return eventHandler();
  } finally {
    currentPriorityLevel = previousPriorityLevel;
  }
}
```

{% endmethod %}
