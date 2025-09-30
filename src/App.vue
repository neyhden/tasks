<script setup>

    import { ref } from "vue"

    const secToFormat = (sec) => {
        const s = sec % 60
        const m = Math.floor(sec / 60) % 60
        const h = Math.floor(sec / 3600)
        return [h,m,s]
            .map(v => v < 10 ? "0" + v : v)
            .join(':')
    }

    let taskId = 1;
    const taskName = ref("")
    const taskEffortHH = ref("")
    const taskEffortMM = ref("")
    const taskEffortSS = ref("")
    const nameref = ref(null)

    /*
        name: String with the name of the task
        effort: Estimated time in seconds
        deadline: deadline for task finish
        notes: Array of strings with task notes
        invested: Time in seconds invested in the task
        interval: Object with the task timer
        active: Wether the task is running or not
    */
    const tasks = ref([
        {
            id: 0,
            name: "Sample task",
            effort: 3600,
            deadline: "2025:10:02:11:00:00",
            notes: ["Sample note 1", "Sample note 2",],
            invested: 0,
            interval: null,
            active: false,
        }
    ])

    const setActiveTask = (task) => {
        activeTask.value = task.id
        task.active = true
    }

    const clickedTask = (task) => {
        task.active = !task.active
        if (task.active) {
            task.interval = setInterval(() => task.invested++, 1000)
        } else {
            clearInterval(task.interval)
        }
    }

    const addTask = () => {
        if (taskName.value === "") return
        let effort = Number(taskEffortHH.value) * 3600
        effort += Number(taskEffortMM.value) * 60
        effort += Number(taskEffortSS.value)
        tasks.value.push({
            id: taskId++,
            name: taskName.value,
            effort: effort,
            deadline: "2025:10:02:11:00:00",
            notes: ["Sample note 1", "Sample note 2",],
            invested: 0,
        })
        taskName.value = ""
        taskEffortHH.value = ""
        taskEffortMM.value = ""
        taskEffortSS.value = ""
        nameref.value.focus()
    }

    const removeTask = (task) => {
        tasks.value = tasks.value.filter(t => t != task)
    }

</script>

<template>

    <TransitionGroup tag="div" name="fade" class="container">
        <div v-for="task in tasks" :key="task.id" class="task" @click="clickedTask(task)">
            <div>
                <span>{{ task.name }}</span>
                <span :class="task.active ? 'running' : 'stopped'">
                    {{ task.active ? "- Running" : "- Stopped" }}
                </span>
            </div>
            <div>
                <span :class="task.invested > task.effort ? 'overtime' : ''">{{ secToFormat(task.invested) }}</span>
                <span class="time-divider">/</span>
                <span :class="task.invested > task.effort ? 'overtime' : ''">{{ secToFormat(task.effort) }}</span>
                <button class="bin" @click="removeTask(task)">
                    <img class="bin-img" src="./bin.svg" alt="X">
                </button>
            </div>
        </div>
        <div class="task">
            <div>
                <input ref="nameref" @keyup.enter="addTask" v-model="taskName" placeholder="New task name">
            </div>
            <div>
                <input size="1" maxlength="2" class="time-input" @keyup.enter="addTask" v-model="taskEffortHH" placeholder="HH">
                <span class="time-divider"> : </span>
                <input size="1" maxlength="2" class="time-input" @keyup.enter="addTask" v-model="taskEffortMM" placeholder="MM">
                <span class="time-divider"> : </span>
                <input size="1" maxlength="2" class="time-input" @keyup.enter="addTask" v-model="taskEffortSS" placeholder="SS">
                <button @click="addTask" class="add-button">+</button>
            </div>
        </div>
    </TransitionGroup>

</template>

<style>

    body {
        background: #000010;
        color: white;
        user-select: none;
        font-size: 18px;
    }

    button {
        background: none;
        box-shadow: none;
        outline: none;
        border: none;
        border-radius: 5px;
        transition-duration: 0.2s
    }
    button:hover {
        background: rgba(255, 255, 255, 0.1);
    }
    button:focus {
        border: 1px solid;
    }

    input {
        background: rgba(255, 255, 255, 0.1);
        border: none;
        outline: none;
        box-shadow: none;
        color: white;
        border-radius: 100px;
        font-size: 24px;
        padding: 0px 20px 0px 20px;
    }

    time-input {
        width: 50px;
    }

    .task {
        background: rgba(255, 255, 255, 0.1);
        transition-duration: 100ms;
        border-radius: 10px;
        margin: 5px 0px 5px 0px;
        padding: 2px 10px 2px 10px;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
    .task:hover {
        background: rgba(255, 255, 255, 0.15);
    }

    .add-button {
        font-weight: bold;
        color: #55ff55;
        font-size: 24px;
        margin: 0px 10px 0px 10px;
    }

    .running {
        margin-left: 20px;
        color: #55ff55;
    }
    .stopped {
        margin-left: 20px;
        color: #ff5555;
    }

    .overtime {
        color: #ff8888;
    }

    .time-divider {
        margin: 0px 20px 0px 20px;
    }

    .bin {
        color: #ff2020;
        margin: 0px 10px 0px 10px;
    }
    .bin-img {
        position: relative;
        top: 2px;
        color: #ff2020;
        height: 20px;
    }

    /* 1. declare transition */
    .fade-move,
    .fade-enter-active,
    .fade-leave-active {
      transition: all 0.3s cubic-bezier(0.55, 0, 0.1, 1);
    }

    /* 2. declare enter from and leave to state */
    .fade-enter-from,
    .fade-leave-to {
      opacity: 0;
      transform: scaleY(0.01) scaleX(0.01);
    }

    /* 3. ensure leaving items are taken out of layout flow so that moving
          animations can be calculated correctly. */
    .fade-leave-active {
      position: absolute;
    }

</style>
