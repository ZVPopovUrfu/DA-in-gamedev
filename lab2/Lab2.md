# Анализ данных в разработке игр
Отчет по лабораторной работе #1 выполнил(а):
- Попов Захар Владимирович
- РИ220943
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.


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

## Цель работы
Научиться передавать в Unity данные из Google Sheets с помощью Python.
## Задание 1
### Выберите одну из компьютерных игр, приведите скриншот её геймплея и краткое описание концепта игры. Выберите одну из игровых переменных в игре (ресурсы, внутри игровая валюта, здоровье персонажей и т.д.), опишите её роль в игре, условия изменения / появления и диапазон допустимых значений. Постройте схему экономической модели в игре и укажите место выбранного ресурса в ней.

Ведьмак 3: Дикая Охота.

![Screnshot]

Компьютерная open world игра action/RPG в сеттинге фэнтези, разработанная и изданная польской студией CD Projekt RED. Изначально игра была выпущена 19 мая 2015 года на Windows, PlayStation 4 и Xbox One, затем 15 октября 2019 года на Nintendo Switch, а 14 декабря 2022 года — на PlayStation 5 и Xbox Series X/S.

Выбрал такую переменную как "наносимый урон". Она зависит от того, какой меч выбран (средний показатель меча = базовый урон), какой типа атаки (быстрая = базовому урону, а мощная = базовый урон * 1.8(3)), крит ли (базовая сила крита 25%, а шанс 20%), уровня персонажа, скиллов, мутагенов, использования зельев... Так как слишком много зависимостей, то я уменишил их и построил экономеческую модель, которую буду реализовывать. 

![Screnshot]

## Задание 2
### С помощью скрипта на языке Python заполните google-таблицу данными, описывающими выбранную игровую переменную в выбранной игре (в качестве таких переменных может выступать игровая валюта, ресурсы, здоровье и т.д.). Средствами google-sheets визуализируйте данные в google-таблице (постройте график, диаграмму и пр.) для наглядного представления выбранной игровой величины.

Генерация данных на Python, заполняющая google-таблицу:
```py

import gspread
import numpy as np
gc = gspread.service_account(filename='unitydatascience-401110-f042e0e7c814.json')
sh = gc.open("Workshop#2")
critical_attacks = np.random.randint(0, 11, 10)
deadly_attacks = np.random.randint(0, 11, 10)
attack_series = [24, 24, 24, 44, 44, 24, 44, 44, 44, 24]
crit = 0
dead = 0
for i in range(0, 10):
    if critical_attacks[i] <= 2:
        crit = 1
    if deadly_attacks[i] <= 1:
        dead = 1
    sh.sheet1.update(('A' + str(i+1)), attack_series[i])
    sh.sheet1.update(('B' + str(i+1)), crit)
    sh.sheet1.update(('C' + str(i+1)), dead)
    crit = 0
    dead = 0

```

Построение диаграммы, для для наглядного представления:
![Screnshot]()

## Задание 3
### Настройте на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной. Например, если выбрано здоровье главного персонажа вы можете выводить сообщения, связанные с его состоянием.

Добавлены звуковый файлы описывающие быструю/мощную атаку, атаку критом или смертельную атаку. 
![Screnshot]()

В консоли выводятся нанесеные уроны, это сопровождается определенным звуком.
![Screnshot]()

Код реализации на C#:
```C#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip quickAttack;
    public AudioClip powerfulAttack;
    public AudioClip critAttack;
    public AudioClip deadlyAttack;
    private AudioSource selectAudio;
    private List<Attack> dataSet = new List<Attack>();
    private bool statusStart = false;
    private int i = 1;
    private float critPower = 1.25f;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet.Count <= i) return;
        if (dataSet[i].dead == 1 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudio(deadlyAttack));
            Debug.Log("Смертельная атака");
        }

        if (dataSet[i].crit == 1 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudio(critAttack));
            Debug.Log(dataSet[i].basicAttack * critPower + " Крит атака");
        }

        if (dataSet[i].basicAttack == 24 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudio(quickAttack));
            Debug.Log(dataSet[i].basicAttack + " Быстрая аттака");
        }

        if (dataSet[i].basicAttack == 44 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudio(powerfulAttack));
            Debug.Log(dataSet[i].basicAttack + " Мощная атака");
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1btlBkjaLx9OtJLb8JK0o73Wf0c_RtRDLtCd-qPdc6K8/values/Лист1?key=AIzaSyCtabUSp8PfgZ_D571iSW71DDh-o69wUrM");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(new Attack(int.Parse(selectRow[0]), int.Parse(selectRow[1]), int.Parse(selectRow[2])));
        }
    }

    IEnumerator PlaySelectAudio(AudioClip audioClip)
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = audioClip;
        selectAudio.Play();
        yield return new WaitForSeconds(2);
        statusStart = false;
        i++;
    }
}

public class Attack
{
    public int basicAttack;
    public int crit;
    public int dead;

    public Attack(int basicAttack, int crit, int dead)
    {
        this.basicAttack = basicAttack;
        this.crit = crit;
        this.dead = dead;
    }
}


```

Репозиторий проекта 

## Выводы

В ходе лабораторной работы я научился передавать данные из таблицы GoogleSheets в Unity с помощью скриптов на языках Python и C#. Научился работать в Unity со звукомыми файлами.

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
