<script setup>
// import location from "../data/location.json";
import temp from "../data/temperatures.json";
// import { ref } from "vue";

// const info = ref({LocoType: temp.LocoType, LocoNumber: temp.LocoNumber})
</script>

<template>
    <div>
        <ul>
            <!-- Альтернативный способ в комментариях -->
            <!-- <li v-for="(item, index) in info" :key="index">
                {{ index + ': ' + item }}
            </li> -->
            <li>
                {{ 'Тип локомотива: ' + temp.LocoType }}
            </li>
            <li>
                {{ 'Номер локомотива: ' + temp.LocoNumber }}
            </li>
        </ul>
    </div>
</template>

<style scoped>
div{
    position: absolute;
    top: 50%;                          
    transform: translate(0, -50%);
    left: 10px;  
    display: flex;
    justify-content: center;
    align-items: center;
    max-width: 250px;
}
ul{
    border: solid 2px rgb(139, 139, 139);
    border-radius: 5px;
    padding: 5px 22px;
    background: rgba(255, 255, 255, 0.74);
    text-align: start;
    color: rgb(100, 100, 100);
    font-weight: bold;
}

</style>