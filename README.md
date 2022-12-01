# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #5 выполнил:
- Батраков Дмитрий Антонович
- НМТ212701
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨


Для выполнения данной лабораторной работы был использован проект Unity, который был создан в рамках лабораторной работы #3 (копия).
Этот проект был изменён, как показано в видео материалах, предоставленых в методических матереалах к данной лабораторной работе.

Была изменена сцена:
![image](https://user-images.githubusercontent.com/113825126/204824936-fdadba79-2fbd-467c-b514-5715a34eb5f0.png)

Был добавлен скрипт Move, предоставленный преподавателями курса
```py
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class Move : Agent
{
    [SerializeField] private GameObject goldMine;
    [SerializeField] private GameObject village;
    private float speedMove;
    private float timeMining;
    private float month;
    private bool checkMiningStart = false;
    private bool checkMiningFinish = false;
    private bool checkStartMonth = false;
    private bool setSensor = true;
    private float amountGold;
    private float pickaxeСost;
    private float profitPercentage;
    private float[] pricesMonth = new float[2];
    private float priceMonth;
    private float tempInf;

    // Start is called before the first frame update
    public override void OnEpisodeBegin()
    {
        // If the Agent fell, zero its momentum
        if (this.transform.localPosition != village.transform.localPosition)
        {
            this.transform.localPosition = village.transform.localPosition;
        }
        checkMiningStart = false;
        checkMiningFinish = false;
        checkStartMonth = false;
        setSensor = true;
        priceMonth = 0.0f;
        pricesMonth[0] = 0.0f;
        pricesMonth[1] = 0.0f;
        tempInf = 0.0f;
        month = 1;
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(speedMove);
        sensor.AddObservation(timeMining);
        sensor.AddObservation(amountGold);
        sensor.AddObservation(pickaxeСost);
        sensor.AddObservation(profitPercentage);
    }

    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        if (month < 3 || setSensor == true)
        {
            speedMove = Mathf.Clamp(actionBuffers.ContinuousActions[0], 1f, 10f);
            Debug.Log("SpeedMove: " + speedMove);
            timeMining = Mathf.Clamp(actionBuffers.ContinuousActions[1], 1f, 10f);
            Debug.Log("timeMining: " + timeMining);
            setSensor = false;
            if (checkStartMonth == false)
            {
                Debug.Log("Start Coroutine StartMonth");
                StartCoroutine(StartMonth());
            }

            if (transform.position != goldMine.transform.position & checkMiningFinish == false)
            {
                transform.position = Vector3.MoveTowards(transform.position, goldMine.transform.position, Time.deltaTime * speedMove);
            }

            if (transform.position == goldMine.transform.position & checkMiningStart == false)
            {
                Debug.Log("Start Coroutine StartGoldMine");
                StartCoroutine(StartGoldMine());
            }

            if (transform.position != village.transform.position & checkMiningFinish == true)
            {
                transform.position = Vector3.MoveTowards(transform.position, village.transform.position, Time.deltaTime * speedMove);
            }

            if (transform.position == village.transform.position & checkMiningStart == true)
            {
                checkMiningFinish = false;
                checkMiningStart = false;
                setSensor = true;
                amountGold = Mathf.Clamp(actionBuffers.ContinuousActions[2], 1f, 10f);
                Debug.Log("amountGold: " + amountGold);
                pickaxeСost = Mathf.Clamp(actionBuffers.ContinuousActions[3], 100f, 1000f);
                Debug.Log("pickaxeСost: " + pickaxeСost);
                profitPercentage = Mathf.Clamp(actionBuffers.ContinuousActions[4], 0.1f, 0.5f);
                Debug.Log("profitPercentage: " + profitPercentage);

                if (month != 2)
                {
                    priceMonth = pricesMonth[0] + ((pickaxeСost + pickaxeСost * profitPercentage) / amountGold);
                    pricesMonth[0] = priceMonth;
                    Debug.Log("priceMonth: " + priceMonth);
                }
                if (month == 2)
                {
                    priceMonth = pricesMonth[1] + ((pickaxeСost + pickaxeСost * profitPercentage) / amountGold);
                    pricesMonth[1] = priceMonth;
                    Debug.Log("priceMonth: " + priceMonth);
                }

            }
        }
        else
        {
            tempInf = ((pricesMonth[1] - pricesMonth[0]) / pricesMonth[0]) * 100;
            if (tempInf <= 6f)
            {
                SetReward(1.0f);
                Debug.Log("True");
                Debug.Log("tempInf: " + tempInf);
                EndEpisode();
            }
            else
            {
                SetReward(-1.0f);
                Debug.Log("False");
                Debug.Log("tempInf: " + tempInf);
                EndEpisode();
            }
        }
    }

    IEnumerator StartGoldMine()
    {
        checkMiningStart = true;
        yield return new WaitForSeconds(timeMining);
        Debug.Log("Mining Finish");
        checkMiningFinish = true;
    }

    IEnumerator StartMonth()
    {
        checkStartMonth = true;
        yield return new WaitForSeconds(60);
        checkStartMonth = false;
        month++;

    }
}
```
Скрипт был добавлен в объект RollerAgent, были внесены изменения в Behavior Parameters согласно методическим материалам.
![image](https://user-images.githubusercontent.com/113825126/204826243-c3d61d1d-455c-4701-a621-bee7a43e2a01.png)

В файлы проекта был добавлен новый yaml-файл с конфигурациями обучения экономической моделии. Файл был предоставлен преподавтелями курса.
```py
behaviors:
  Economic:
    trainer_type: ppo
    hyperparameters:
      batch_size: 1024
      buffer_size: 10240
      learning_rate: 3.0e-4
      learning_rate_schedule: linear
      beta: 1.0e-2
      epsilon: 0.2
      lambd: 0.95
      num_epoch: 3    
    network_settings:
      normalize: true
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    checkpoint_interval: 500000
    max_steps: 750000
    time_horizon: 64
    summary_freq: 5000
    self_play:
      save_steps: 20000
      team_change: 100000
      swap_steps: 10000
      play_against_latest_model_ratio: 0.5
      window: 10
```

Модель была успешно обучена, результаты обучения были обработаны и представлены в виде графиков в tensorboard.
![image](https://user-images.githubusercontent.com/113825126/204828809-672b1604-942a-4caf-a021-b3ae78cedc81.png)
![image](https://user-images.githubusercontent.com/113825126/204828876-00df0af5-e63e-4bc8-8c05-3518366123eb.png)

При данной конфигурации, вознаграждение в процессе обучения оставлось постоянным, потери "Политики"(курса) и потери значений (стоимости) постепенно уменьшались.

## Задание 1
### Измените параметры файла. yaml-агента и определить какие параметры и как влияют на обучение модели.

1) 
```py
      batch_size: 1524
      buffer_size: 15240
```
batch_size должен быть в несколько раз меньше чем buffer_size, поэтому изменяю их одновременно и в одинаковой пропорции.
![image](https://user-images.githubusercontent.com/113825126/204839141-787aefea-83ba-490c-926b-643c4a801348.png)
![image](https://user-images.githubusercontent.com/113825126/204839197-98ac6138-0507-4af4-8333-5c88c5c5229f.png)

Изменения по сравнению с исходной конфигурацией:

Cumulative Reward	        Нет изменений

Policy Loss		            Начальное значение немного ниже, держится на постоянном уровне

Value Loss	                Конечное значение ниже

Beta	                    Уменьшение нзначительно

Entropy	                    Незначительные отклонения

Epsilon	                    Оставался постоянным

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение выше

Learning Rate               Оставался постоянным

Обучение стало более стабильным, в первую очередь это связано с увеличением buffer_size.

2) 
```py
      learning_rate: 3.0e-3
```

![image](https://user-images.githubusercontent.com/113825126/204840938-8b6df8b7-323c-4549-88d7-cd6b678f7b33.png)
![image](https://user-images.githubusercontent.com/113825126/204840978-c97a2cb3-8703-4ce7-b4d1-1cc4e33f0e07.png)

Cumulative Reward	        Начальное значение немного выше

Policy Loss		            Начальное значение выше

Value Loss	                График и конечное значение выше

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Начальное значение выше

Extrinsic Value Estimate	График и конечное значение выше

Learning Rate               График и конечное значение ниже

Обычно это значение следует уменьшать, если обучение нестабильно, а вознаграждение не увеличивается постоянно. 
В нашем случае, при увеличении learning_rate, потери стали больше, обучение стало менее стабильным.

3) 
```py
      learning_rate: 3.0e-5
```
![image](https://user-images.githubusercontent.com/113825126/204844454-17df11c0-5a1e-4cfa-b6f7-e8d38f109603.png)
![image](https://user-images.githubusercontent.com/113825126/204844478-91f5e170-4b8f-4294-bf5a-a899878bdb95.png)

Cumulative Reward	        Нет изменений

Policy Loss		            Конечное значение выше

Value Loss	                Незначительное отклонение

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение ниже

Learning Rate               Нет изменений

По графикам нельзя сказать, что при уменьшении learning_rate в нашем случае обучение стало более стабильным.

4) 
```py
    beta: 1.0e-3
```
![image](https://user-images.githubusercontent.com/113825126/204846065-5723aad0-dd9b-4cf3-a48a-ddcc666ae5f8.png)
![image](https://user-images.githubusercontent.com/113825126/204846094-db5c6792-131d-414a-9565-c5c413d1968b.png)

Cumulative Reward	        Нет изменений

Policy Loss		            График выше

Value Loss	                График значительно ниже

Beta	                    График и конечное значение выше

Entropy	                    Возрастает

Epsilon	                    Отклонение

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График выше, ломанный вид

Learning Rate               График ниже

beta уменьшен, обучение стало менее стабильным.

5) уменьшим beta ещё раз
```py
    beta: 1.0e-4
```
![image](https://user-images.githubusercontent.com/113825126/204853544-5c182a62-76fc-4300-99b7-a9562e24bd92.png)
![image](https://user-images.githubusercontent.com/113825126/204853575-da23af91-a07f-4e5a-8e1f-82927b01303b.png)

Cumulative Reward	        Начальное значение выше (возможно, случайность)

Policy Loss		            Возрастает, потом убывает, конечное значение ниже

Value Loss	                Зигзагообразный график, конечное значение значительно ниже

Beta	                    График и конечное значение ниже

Entropy	                    Возрастает (почти линейно)

Epsilon	                    Отклонение

Extrinsic Reward	        Начальное значение выше (возможно, случайность)

Extrinsic Value Estimate	График выше, ломанный вид

Learning Rate               Отклонение

beta ещё меньше, потери политики и стоимости стали меньше

6) 
```py
      epsilon: 0.1
```

Cumulative Reward	        Нет изменений

Policy Loss		            Небольшое отклонение, конечное значение немного выше

Value Loss	                График и конечное значение значительно выше

Beta	                    Нет изменений

Entropy	                    Небольшое отклонение

Epsilon	                    Остаётся постоянным

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение ниже

Learning Rate               Отклонение

epsilon соответствует допустимому порогу расхождения между старой и новой политикой при обновлении градиентного спуска.
Параметр уменьшен, потери значиетльно возрасли

7) 
```py
      epsilon: 0.3
```

Cumulative Reward	        Нет изменений

Policy Loss		            График выше

Value Loss	                Небольшое отклонние, конечное значение то же

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    График и конечное значение выше

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение выше, возрастает, потом убывает

Learning Rate               Нет изменений

epsilon увеличен, нельзя сказать, что обучение стало стабильнее

8) 
```py
      lambd: 0.9
```

Cumulative Reward	        Нет изменений

Policy Loss		            Возрастает, потом убывает, конечное значение выше

Value Loss	                График и конечное значение ниже

Beta	                    Нет изменений

Entropy	                    Убывает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение выше

Learning Rate               Нет изменений

Потери значений стали меньше, Потери курса увеличились незначительно, обученеи стало стабильнее.

lambd уменьшен, нельзя сказать, что обучение стало стабильнее.

9) Параметр lambd можно рассматривать как то, насколько агент полагается на свою текущую оценку стоимости при вычислении обновленной оценки стоимости. Рекомендуется перебирать различные значения, чтобы найти то, при котором обучение бедет стабльнее. Значения 0,9 и 0,95 - границы стандартного диапазона значений, попробуем середину диапазона.
```py
      lambd: 0.93
```

Cumulative Reward	        Нет изменений

Policy Loss		            Возрастает, потом убывает, конечное значение выше

Value Loss	                График и конечное значение ниже

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение выше, возрастает

Learning Rate               Нет изменений

Потери значений стали меньше, Потери курса увеличились незначительно, можно сказать, что обученеи стало стабильнее.

10) num_epoch - количество проходов через буфер опыта при выполнении оптимизации градиентного спуска. Чем больше размер партии, тем больше это допустимо. Уменьшение этого параметра обеспечит более стабильные обновления за счет более медленного обучения.
```py
      num_epoch: 10
```

Cumulative Reward	        Нет изменений

Policy Loss		            График немного выше, конечное значение почти не изменилось

Value Loss	                График и конечное значение ниже

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График выше и конечное значение выше, возрастает, потом убывает

Learning Rate               Нет изменений

num_epoch увеличен, Потери Значений стали меньше, можно сказать, что обученеи стало стабильнее

11)
```py
        normalize: true
```

Cumulative Reward	        начальное значение выше

Policy Loss		            График выше

Value Loss	                График ниже

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        начальное значение выше

Extrinsic Value Estimate	График выше, убывает, потом убывает

Learning Rate               Нет изменений

Нельзя сказать, что обучение стало стабильнее

12)
hidden_units - Количество единиц в скрытых слоях нейронной сети. Соответствуют количеству единиц в каждом полносвязном слое нейронной сети. Для простых задач, где правильное действие представляет собой простую комбинацию входных данных наблюдения, это значение должно быть небольшим. Для задач, где действие представляет собой очень сложное взаимодействие между переменными наблюдения, это значение должно быть больше.

```py
      hidden_units: 64
```
![image](https://user-images.githubusercontent.com/113825126/205047389-3a76b27f-131b-43f9-928b-22f61cf47cc2.png)
![image](https://user-images.githubusercontent.com/113825126/205047403-7b46c56f-4862-4c70-ad57-05adae4b2185.png)


Cumulative Reward	        начальное значение выше

Policy Loss		            График немного выше

Value Loss	                График и конечое значение выше

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График выше и конечное значение выше, возрастает, потом убывает

Learning Rate               Нет изменений

hidden_units уменьшен в 2 раза, обучение стало менее стабильным, значит, возможно, в нашй сети присутствуют сложные взаимодействия, увеличим параметр.

13)
```py
      hidden_units: 256
```


Cumulative Reward	        Нет изменений

Policy Loss		            График выше

Value Loss	                График и конечое значение немного выше

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение немного ниже

Learning Rate               Нет изменений

hidden_units увеличен в 2 раза, обучение не стало стабильнее по сравнению с исходной конфигурацией.

14)
```py
      num_layers: 3
```

Cumulative Reward	        Нет изменений

Policy Loss		            График и конечное значение немного выше, убывает, потом возрастает

Value Loss	                График и конечое значение выше

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение выше, возрастает, потом убывает

Learning Rate               Нет изменений

Количество скрытых слоев в нейронной сети. Соответствует количеству скрытых слоев после ввода наблюдения или после кодирования CNN визуального наблюдения. Для простых задач меньше слоев, скорее всего, будут обучать быстрее и эффективнее. Для более сложных задач управления может потребоваться больше слоев.
num_layers увеличен, обучение стало менее стабильным.

15)
```py
      num_layers: 1
```

Cumulative Reward	        Нет изменений

Policy Loss		            Концы графика имею едва заметный наклон (убывает, потом возрастает), конечное значение немного выше

Value Loss	                График и конечое значение выше

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение выше, возрастает, потом убывает

Learning Rate               Нет изменений

num_layers уменьшен, нельзя сказать, что обучение стало стабильнее.

16)
```py
        gamma: 0.8
```

Cumulative Reward	        Нет изменений

Policy Loss		            Убывает

Value Loss	                График и конечое значение выше

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение выше, возрастает, потом убывает

Learning Rate               Нет изменений

Фактор скидки для будущих вознаграждений, поступающих из окружающей среды. Это можно рассматривать как то, как далеко в будущем агент должен заботиться о возможных вознаграждениях. В ситуациях, когда агент должен действовать в настоящем, чтобы подготовиться к вознаграждению в отдаленном будущем, это значение должно быть большим. В случаях, когда вознаграждение является более немедленным, оно может быть меньше. Должен быть строго меньше 1.
gamma уменьшен, обучение не стало стабильнее, возможно, это связано с тем, что модель "редко" получает вознаграждение, увеличим параметр.

17)
```py
        gamma: 0.995
```

Cumulative Reward	        Начальное значение выше (случайность)

Policy Loss		            График и конечое значение выше, возрастает, потом убывает

Value Loss	                График и конечое значение выше, возрастает, потом убывает

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Начальное значение выше (случайность)

Extrinsic Value Estimate	График и конечное значение выше, возрастает, потом убывает

Learning Rate               Нет изменений

gamma увеличен, обучение нестало стабильнее


18)
```py
        strength: 5.0
```

Cumulative Reward	        Нет изменений

Policy Loss		            Убывает

Value Loss	                График и конечое значение немного выше

Beta	                    Нет изменений

Entropy	                    Небольшое отклонение

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение немного выше

Learning Rate               Нет изменений

Коэффициент, на который умножается вознаграждение, данное средой. Типичные диапазоны будут варьироваться в зависимости от сигнала вознаграждения.
strength увеличен в 5 раз, нелзьзя сказать, что обучение стало стабильнее.

19)
```py
        strength: -1.0
```

Cumulative Reward	        Нет изменений

Policy Loss		            Убывает

Value Loss	                График и конечое значение выше

Beta	                    Нет изменений

Entropy	                    Ломанная кривая линия

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение немного ниже

Learning Rate               Нет изменений

Изменение strength слабо (или почти не) влияет на стабильность обучения, внешнее вознагараждение стало положительным.

19) Количество опыта, полученного тренером между каждой контрольной точкой.
```py
    checkpoint_interval: 750000
```

Cumulative Reward	        Нет изменений

Policy Loss		            Убывает

Value Loss	                График и конечое значение немного выше

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение выше, возрастает, потом убывает

Learning Rate               Нет изменений

Количество опыта, полученного тренером между каждой контрольной точкой. checkpoint_interval увеличен, почти не влияет на стабильность обучения.

20)
```py
        max_steps: 500000
```

Cumulative Reward	        Нет изменений

Policy Loss		            Возрастает, конечое значение выше

Value Loss	                График и конечое значение выше

Beta	                    Нет изменений

Entropy	                    Возрастает

Epsilon	                    Нет изменений

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение выше, возрастает, потом убывает

Learning Rate               Нет изменений

 Общее количество шагов (т. е. собранных наблюдений и предпринятых действий), которые необходимо выполнить в среде (или во всех средах при параллельном использовании нескольких) перед завершением процесса обучения. max_steps уменьшен, обучение не стало стабильнее.
 
 21)
```py
    time_horizon: 128
```

Cumulative Reward	        Нет изменений

Policy Loss		            Возрастает, потом убывает, конечое значение выше

Value Loss	                График и конечое значение выше

Beta	                    График и конечое значение ниже

Entropy	                    Возрастает

Epsilon	                    Убывает

Extrinsic Reward	        Нет изменений

Extrinsic Value Estimate	График и конечное значение выше, возрастает, потом убывает

Learning Rate               Убывает

Сколько шагов опыта необходимо собрать для каждого агента, прежде чем добавить его в буфер опыта. В тех случаях, когда в эпизоде есть частые награды или эпизоды непомерно велики, более идеальным может быть меньшее количество. Это число должно быть достаточно большим, чтобы охватить все важные действия в последовательности действий агента. time_horizon увеличен в двое, обучение стало менее стабильным. Уменьшим параметр

## Задание 2
### Опишите результаты, выведенные в TensorBoard

## Выводы
### Абзац умных слов о том, что было сделано и что было узнано.

В ходе данной лабораторной работы я узнал, что такое перцептрон, как он работает, почему разделяют машинное и глубокое обучение, что такое XOR и NAND,
обучил перцептрон выполнять вычисления OR, AND, NAND, проанализировал его работу, оценил его возможности, познакомился с процессом обучения перцептрона.

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
