<script setup>
import { YandexMap } from "vue-yandex-maps";
import { markRaw } from "vue";
import LocoInfo from "./LocoInfo.vue";

import location from "../data/location.json";
import temp from "../data/temperatures.json";

const locationsArray = markRaw([]);
const tempArray = markRaw([]);

// Форматирование времени
// function spreadTimestamp(date){
//     let obj ={
//         hours: [],
//         minutes: [],
//         seconds: []
//     }
//     for (let i = 0; i < date.length; i++){
//     let time = date[i].split(" ")[1]; // "10:00:00"
//     let hours = time.substring(0, 2); // "10"
//     let minutes = time.substring(3, 5); // "00"
//     let seconds = time.substring(6, 8); // "00"
//     obj.hours.push(hours)
//     obj.minutes.push(minutes)
//     obj.seconds.push(seconds)
//     }
// }

function timeCompare(i, k) {}

// Форматирование координат в массив [длина, широта]
for (let i = 0; i < location.Latitude.length; i++) {
    let point = [];
    if (
        typeof location.Latitude[i] != "string" ||
        typeof location.Longitude[i] != "string"
    ) {
        point.push(location.Latitude[i], location.Longitude[i]);
        locationsArray.push(point);
        console.log(typeof location.Latitude[i], typeof location.Longitude[i]);
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
            strokeStyle: "2  2  5",
        }
    );

    // Создадим коллекцию меток
    const placemarksArray = markRaw([]);
    const placemarks = new ymaps.GeoObjectCollection();
    for (let i = 0; i < locationsArray.length; i++) {


        // ЕСЛИ ХВАТИТ СИЛ ДЕКОМПОЗИРУЮ ВСЁ
        // ТАКЖЕ БЫЛО БЫ ХОРОШО УСРЕДНИТЬ ВРЕМЯ ЛОКОМОТИВА ДЛЯ ОТОБРАЖЕНИЯ БОЛЕЕ ТОЧНОЙ ТЕМПЕРАТУРЫ


        // Определение температуры
        const dateTemp = (k) => new Date(`${temp.Timestamp[k]}`);
        const dateLoco = new Date(`${location.Timestamp[i]}`);
        let currentTemp
        if (dateLoco > dateTemp(0) && dateLoco < dateTemp(1)){
            currentTemp = temp.OutsideTemp[0]
        } else if (dateLoco > dateTemp(1) && dateLoco < dateTemp(2)) {
            currentTemp = temp.OutsideTemp[1]
        } else if (dateLoco > dateTemp(2) && dateLoco < dateTemp(3)) {
            currentTemp = Math.round((temp.OutsideTemp[2] + temp.OutsideTemp[3])/2)
        } else if (dateLoco >= dateTemp(3)){
            currentTemp = temp.OutsideTemp[3]
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
