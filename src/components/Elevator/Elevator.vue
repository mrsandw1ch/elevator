<template lang="">
    <div id="elevator">

        <div id="floors">
            <div
                class="floor"
                v-for="n in floorsCount"
                :key="'floor-' + n">
            </div>
        </div>

        <div id="buttons">
            <button
                v-for="n in floorsCount"
                :key="'call-button-' + n"
                :style="{backgroundColor: this.shafts.find(shaft => {
                    return shaft.calls.indexOf(n) > -1 && shaft.status !== 'idle'
                }) ? 'red' : 'transparent'}"
                @click="call(n)"
                >{{ n }}
            </button>
        </div>
        
        <div id="shafts">
            <Shaft
                v-for="shaft in shafts"
                :key="'shaft-' + shaft.id"
                :id="shaft.id"
                :currentFloor="shaft.calls[0]"
                :status="shaft.status"
                :travelTime="shaft.travelTime"
            />
        </div>
    </div>
</template>

<script>
import Shaft from './components/Shaft.vue'

const floorsCount = 8;
const shaftsCount = 4;

export default {
    name: 'Elevator',
    components: {
        Shaft
    },

    data() {
        return {
            floorsCount: floorsCount,
            shaftsCount: shaftsCount,
            /* 
                Создаем массив лифтовых шахт в количестве shaftsCount
                По умолчанию лифт в каждой из них свободен и стоит на 1-м этаже
                travelTime - вспомогательное свойство, передается в компонент Shaft и устанавливает длительность анимации
            */
            shafts: Array.from({ length: shaftsCount }, (_, i) => ({
                id: i,
                calls: [1],
                status: 'idle',
                travelTime: 0
            }))

        }
    },

    // Загружаем сохраненное состояние из localStorage, если оно есть
    mounted() {
        if (localStorage.shafts) {
            this.shafts = JSON.parse(localStorage.shafts)

            // После перезагрузки страницы лифт "отдыхает" 3 секунды на том этаже, который обрабатывал
            this.shafts.forEach(shaft => {
                if (shaft.status !== 'idle') {
                    this.rest(shaft.id)
                }
            })
        }
    },

    // Отслеживаем и сохраняем все изменения в localStorage
    watch: {
        shafts: {
            handler(newShafts) {
                localStorage.shafts = JSON.stringify(newShafts)
            },
            deep: true
        }
    },

    // Чистим localStorage перед размонтированием компонента
    beforeUnmount() {
        localStorage.clear()
    },

    methods: {
        // ВЫЗОВ ЛИФТА на этаж floor
        call(floor) {
            // Проверяем, что вызов не обрабатывается уже каким-либо из лифтов
            if (this.shafts.find(shaft => shaft.calls.indexOf(floor) > -1) === undefined) {
                // Находим подходящий лифт для обработки вызова
                const targetId = this.shafts.reduce((prevShaft, currShaft) => {
                    // Из лифтов с одинаковыми по длине очередями вызовов выбираем единственный свободный
                    // Если все свободны или заняты - выбираем ближайший или тот, который окажется ближайшим к целевому этажу
                    if (prevShaft.calls.length === currShaft.calls.length) {
                        return prevShaft.status === 'idle' && currShaft.status !== 'idle' ? prevShaft
                             : prevShaft.status !== 'idle' && currShaft.status === 'idle' ? currShaft
                             : Math.abs(currShaft.calls.slice(-1) - floor) < Math.abs(prevShaft.calls.slice(-1) - floor) ? currShaft : prevShaft
                    // В остальных случаях выбираем лифт с наиболее короткой очередью вызовов
                    } else {
                        return prevShaft.calls.length < currShaft.calls.length ? prevShaft : currShaft
                    }
                }).id
                // Отправляем вызов найденному лифту
                this.shafts[targetId].calls.push(floor)
                // И если он свободен, приводим его в движение
                if (this.shafts[targetId].status === 'idle') {
                    this.move(targetId)
                }
            }
        },

        // ДВИЖЕНИЕ ЛИФТА по шахте с заданным id
        move(shaftId) {
            let distance = this.shafts[shaftId].calls[1] - this.shafts[shaftId].calls[0]
            // Определяем время в пути (в мс) и направление движения
            this.shafts[shaftId].travelTime = Math.abs(distance) * 1000
            this.shafts[shaftId].status = (distance > 0 ? 'up' : 'down')
            // Лифт покидает текущий этаж, выбрасывая его из очереди вызовов
            this.shafts[shaftId].calls.shift()
            // По приезде лифт "отдыхает"
            setTimeout(() => {
                this.rest(shaftId)
            }, this.shafts[shaftId].travelTime)
        },

        // "ОТДЫХ" ЛИФТА в шахте с заданным id
        rest(shaftId) {
            this.shafts[shaftId].status = 'rest'
            // Через 3 секунды "отдыха" лифт либо продолжает движение, либо свободен
            setTimeout(() => {
                if (this.shafts[shaftId].calls.length > 1) {
                    this.move(shaftId)
                } else {
                    this.shafts[shaftId].status = 'idle'
                }
            }, 3000)
        }
    }
}
</script>

<style lang="scss">
#elevator {
    position: relative;
    margin: auto;
    margin-top: 120px;
    width: calc(#{$shaft-width} * v-bind('shaftsCount') + 100px);

    .floor {
        border: 1px solid;
        height: $floor-height;
    }

    #buttons {
        position: absolute;
        top: 0;
        left: 0;
        display: flex;
        flex-direction: column-reverse;
        justify-content: space-around;
        height: 100%;
        width: 30px;
        margin-left: 35px;
    }

    #shafts {
        position: absolute;
        top: 0;
        left: 100px;
        height: 100%;
        display: flex;
        flex-direction: row;
    }
}
</style>