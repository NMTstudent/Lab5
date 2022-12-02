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

Модель была успешно обучена, для этого потребовалось около 125000 шагов обучения, результаты обучения были обработаны и представлены в виде графиков в tensorboard.
![image](https://user-images.githubusercontent.com/113825126/205319925-50bdcd81-14f6-40dc-802e-0984b359cdb9.png)


## Задание 1
### Измените параметры файла. yaml-агента и определить какие параметры и как влияют на обучение модели.

В ходе выполнения этого задания некоторые параметры файла. yaml-агента, в некоторых случаях тесносвязанные параметры изменялись парами, остальные параметры возращались к первоначальным значениям.

1)      ```py 
            batch_size: 2048
            buffer_size: 20480
         ```
![image](https://user-images.githubusercontent.com/113825126/205318170-ce1eea2f-159a-4fad-9731-96f26f0dacf3.png)

График Value Loss стал выше, график Policy Loss ниже. 
Обучение проходило более эффективно по сравнению с исходной конфигурацией.

2)      ```py 
            learning_rate: 3.0e-5
         ```
![image](https://user-images.githubusercontent.com/113825126/205322305-2ce40647-99fa-45d8-9f02-70bf8f11d25a.png)

График Value Loss стал ниже, график Policy Loss выше. 
Обучение проходило менее эффективно по сравнению с исходной конфигурацией.

3)      ```py 
            beta: 1.0e-4
         ```
![image](https://user-images.githubusercontent.com/113825126/205325480-e63beb23-8e05-4585-9f2c-800b6c623a10.png)

График Value Loss стал ниже, чтобы Cumulative Reward достиг 1 потребовалось больше ходов. 
Обучение проходило менее эффективно по сравнению с исходной конфигурацией.

4)      ```py 
            hidden_units: 256
       ```
![image](https://user-images.githubusercontent.com/113825126/205326666-519556b8-2912-4c25-b541-5c6d3b583b85.png)

График Value Loss остался на том же уровне, чтобы Cumulative Reward достиг 1 потребовалось меньше ходов. 
Обучение проходило менее эффективно по сравнению с исходной конфигурацией.

5)      ```py 
            gamma: 0.5
       ```
![image](https://user-images.githubusercontent.com/113825126/205329888-76d94f6e-222f-4ad6-a428-6c801e69c19d.png)

График Value Loss стал ниже, чтобы Cumulative Reward достиг 1 потребовалось меньше ходов.
Обучение проходило менее эффективно по сравнению с исходной конфигурацией.

6)      ```py 
            save_steps: 40000
            team_change: 200000
       ```
![image](https://user-images.githubusercontent.com/113825126/205331849-a91b4858-0921-46cf-8a77-125a8c64fea6.png)

График Value Loss стал ниже, График Episode Length стал выше и ломанным, Cumulative Reward не достиг 1 даже за 150000 шагов, График Policy Loss стал ломанным.
Обучение проходило менее эффективно по сравнению с исходной конфигурацией.

## Задание 2
### Опишите результаты, выведенные в TensorBoard

График Cumulative Reward показывает среднюю совокупную награду за эпизод по всем агентам (RollerAgent и Economics). 
Значение должно увеличиваться во время успешной тренировки.

![image](https://user-images.githubusercontent.com/113825126/205304129-a9b50a6a-2984-405c-8290-e1e079a82234.png)

По вертикали значения среднегей совокупной награды, по горизонтали количество шагов обучения.

График Episode Length показывает среднюю продолжительность каждого эпизода в среде для всех агентов.

![image](https://user-images.githubusercontent.com/113825126/205305004-e8eba44e-b517-4819-bb2a-17ac8d79e2d2.png)

По вертикали средняя продолжительность эпизода, по горизонтали количество шагов обучения.

График Policy Loss показывает среднюю величину функции потерь курса (политики). 
Соотносится с тем, насколько сильно меняется политика (процесс принятия решений). 
Эта величина должна уменьшаться во время успешной тренировки.

![image](https://user-images.githubusercontent.com/113825126/205305862-c4556d83-26f0-4d01-9596-6c86c2b91117.png)

По вертикали значения средней величины функции потерь курса (политики), по горизонтали количество шагов обучения.

График Value Loss показывает средняя потерю обновления функции значения. 
Value Loss коррелирует с тем, насколько хорошо модель способна предсказать значение каждого состояния. 
Это величина должна увеличиваться, пока агент обучается, а затем уменьшаться, когда вознаграждение стабилизируется.

![image](https://user-images.githubusercontent.com/113825126/205307059-6a598d30-120a-4072-a0d3-75efc65f9827.png)

По вертикали значения средней потери обновдления функции значения, по горизонтали количество шагов обучения.

## Выводы
### Абзац умных слов о том, что было сделано и что было узнано.

В ходе данной лабораторной работы я узнал, как можно наблюдать обучение ML агента при помощи TensorBoard, что показывают графики в TensorBoard, узнал подробнее об обучении ML агента, провёл эксперементы с файлом конфигурации ML агента, наблюдал и анализировал результаты обучения ML агента в TensorBoard, опытным путём нашёл параметры в файле конфигурации, изменение которых делает обучение ML агента эффективнее.

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
