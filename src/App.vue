<script setup>
import { YandexMap, YandexMarker, loadYmap } from "vue-yandex-maps";
import Map from './components/Map.vue'

const settings = {
    apiKey: "c96ebca7-fd8f-448a-be11-ece82e9b31af", // Индивидуальный ключ API
    lang: "ru_RU", // Используемый язык
    coordorder: "latlong", // Порядок задания географических координат
    debug: false, // Режим отладки
    version: "2.1", // Версия Я.Карт
};




const startPoint = [59.13262376, 37.84800256];
const endPoint = [59.13253209, 37.84717521];
</script>

<template>
    <!-- <Suspense> -->
        <Map> </Map>
    <!-- </Suspense> -->
</template>

<style scoped></style>
