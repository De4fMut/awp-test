<script setup>
import { YandexMap } from "vue-yandex-maps";
import { markRaw } from "vue";
import LocoInfo from "./LocoInfo.vue";

import location from "../data/location.json";
import temp from "../data/temperatures.json";

const locationsArray = markRaw([]);

// Форматирование времени
// function calculateTemperature(date) {
//   const temp10 = temp.OutsideTemp[0];
//   const temp11 = temp.OutsideTemp[1];
//   const temp1155 = temp.OutsideTemp[2];
//   const temp21 = temp.OutsideTemp[3];
//   const time = date.split(' ')[1];

//   console.log(time)

//   const hour = parseInt(time.split(":")[0]);
//   const minute = parseInt(time.split(":")[1]);

// console.log(hour, '  ', minute)

//   const earlierTime = hour < 11 || (hour === 11 && minute <= 55) ? "11:55" : "21:00";
//   const laterTime = hour >= 11 && minute >= 55 ? "21:00" : "10:00";

//   console.log(earlierTime, laterTime)

//   const earlierTemp = earlierTime === "10:00" ? temp10 : temp1155;
//   const laterTemp = laterTime === "21:00" ? temp21 : temp1155;

//   const timeDiff = (hour - parseInt(earlierTime.split(":")[0])) * 60 + (minute - parseInt(earlierTime.split(":")[1]));
//   const tempDiff = laterTemp - earlierTemp;

//   const temperature = earlierTemp + (tempDiff * timeDiff) / ((laterTime === "21:00" ? 24 : 10) * 60 - parseInt(earlierTime.split(":")[0]) * 60 - parseInt(earlierTime.split(":")[1]));

//   alert(`Temperature at ${time} is ${temperature}°C`);
// }

// calculateTemperature(temp.Timestamp[3]);
function roundedTime(t) {
    let date = new Date(t); // дата
    let minutes = date.getMinutes(); // получаем минуты
    let sec = date.getSeconds(); // получаем секунды
    let hours = date.getHours(); // получаем часы

    // округляем секунды до ближайшего целого
    sec = Math.round(sec / 60) * 60;

    if (sec === 60) {
        minutes++;
        sec = 0;
    }

    // округляем минуты до ближайшего целого
    minutes = Math.round(minutes / 60) * 60;

    // если минуты округлились до 60, то увеличиваем часы на 1
    if (minutes === 60) {
        hours++;
        minutes = 0;
    }

    const roundedTime = new Date(
        t.split(" ")[0] +
            " " +
            hours +
            ":" +
            (minutes < 10 ? "0" : "") +
            minutes +
            ":" +
            (minutes < 10 ? "0" : "") +
            sec
    ).getTime();

    // console.log(roundedTime);
    return roundedTime;
}
roundedTime(temp.Timestamp[2]);


// Форматирование координат в массив [длина, широта]
for (let i = 0; i < location.Latitude.length; i++) {
    let point = [];
    if (
        typeof location.Latitude[i] != "string" ||
        typeof location.Longitude[i] != "string"
    ) {
        point.push(location.Latitude[i], location.Longitude[i]);
        locationsArray.push(point);
        // console.log(typeof location.Latitude[i], typeof location.Longitude[i]);
    } else {
        i++;
    }
}

// Параметры карты
const settings = {
    apiKey: "c96ebca7-fd8f-448a-be11-ece82e9b31af", // Индивидуальный ключ API
    lang: "ru_RU", // Используемый язык
    coordorder: "latlong", // Порядок задания географических координат
    debug: false, // Режим отладки
    version: "2.1", // Версия Я.Карт
};

// Получение инстанса данной карты и дальнейшее его использование
function onInit(e) {
    const mapInstance = e;
    const myPolyline = new ymaps.Polyline(
        locationsArray,
        {},
        {
            strokeWidth: 3,
            strokeStyle: "3  2  5",
        }
    );

    // Создадим коллекцию меток
    const placemarksArray = markRaw([]);
    const placemarks = new ymaps.GeoObjectCollection();
    for (let i = 0; i < locationsArray.length; i++) {
        // Можно декомпозировать
        

        // Массив температуры
        const dateTemp = [
            roundedTime(temp.Timestamp[0]),
            roundedTime(temp.Timestamp[1]),
            roundedTime(temp.Timestamp[2]),
            roundedTime(temp.Timestamp[3]),
        ];
        const dateLoco = roundedTime(location.Timestamp[i]);
        // console.log(dateLoco, ': ', dateTemp[0], '= ', dateLoco == dateTemp[0]);

        // Присвоение температуры
        let currentTemp;
        if (dateLoco == dateTemp[0]) {
            currentTemp = temp.OutsideTemp[0];
        } else if (dateLoco == dateTemp[1]) {
            currentTemp = temp.OutsideTemp[1];
        } else if (dateLoco == dateTemp[2]) {
            currentTemp = temp.OutsideTemp[3];
        } else if (dateLoco == dateTemp[3]) {
            currentTemp = temp.OutsideTemp[3];
        }

        // Создание точки
        placemarksArray.push(locationsArray[i]);
        placemarks.add(
            new ymaps.Placemark(
                placemarksArray[i],
                {
                    balloonContent: `<ul>
                        <li>
                            Примерная температура снаружи: ${currentTemp} &#8451;
                        </li>
                        <li>
                            Локомотив был тут в: ${location.Timestamp[i]}
                        </li>
                    </ul>`,
                    iconContent: `${i + 1}`,
                },
                {
                    preset: "islands#violetCircleIcon",
                    // Отключаем кнопку закрытия балуна.
                    balloonCloseButton: false,
                    // Балун будем открывать и закрывать кликом по иконке метки.
                    hideIconOnBalloonOpen: false,
                }
            )
        );
    }

    // Подключение всех созданных объектов к карте
    mapInstance.geoObjects.add(placemarks);
    mapInstance.geoObjects.add(myPolyline);
    mapInstance.setBounds(myPolyline.geometry.getBounds());
    mapInstance.controls.remove("geolocationControl");
    mapInstance.controls.remove("trafficControl");
    mapInstance.controls.remove("fullscreenControl");

    // Вешаем событие
    myPolyline.events.add("mouseenter", function (e) {
        const cursorCoords = e.get("coords");
        const nearest = ymaps.geoQuery(placemarks).getClosestTo(cursorCoords);
        nearest.balloon.open();
        console.log(nearest.geometry._coordinates);
    });
}
</script>

<template>
    <YandexMap
        @created="onInit"
        style="width: 100vw; height: 100vh"
        :settings="settings"
        :coordinates="locationsArray[0]"
    >
    </YandexMap>
    <LocoInfo></LocoInfo>
</template>

<style scoped></style>
