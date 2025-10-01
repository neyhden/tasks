<script setup>

    import { ref, onMounted } from "vue"

    const secToFormat = (sec) => {
        const s = sec % 60
        const m = Math.floor(sec / 60) % 60
        const h = Math.floor(sec / 3600)
        return [h,m,s]
            .map(v => v < 10 ? "0" + v : v)
            .join(':')
    }

    const taskName = ref("")
    const taskEffortHH = ref("")
    const taskEffortMM = ref("")
    const taskEffortSS = ref("")
    const nameref = ref(null)

    let noteId = 0

    let alertId = 0
    const alerts = ref([])

    /*
        name: String with the name of the task
        effort: Estimated time in seconds
        deadline: deadline for task finish
        notes: Array of strings with task notes
        invested: Time in seconds invested in the task
        interval: Object with the task timer
        active: Wether the task is running or not
    */
    let taskId = 0;
    const tasks = ref([ ])

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
        if (taskName.value === "") {
            addAlert("Insert task name")
            return
        }
        if (taskEffortHH.value === "" && taskEffortMM.value === "" && taskEffortSS.value === "") {
            addAlert("Add some estimated effort")
            return
        }
        let effort = Number(taskEffortHH.value) * 3600
        effort += Number(taskEffortMM.value) * 60
        effort += Number(taskEffortSS.value)
        tasks.value.push({
            id: taskId++,
            name: taskName.value,
            effort: effort,
            deadline: "2025:10:02:11:00:00",
            note: "",
            notes: [],
            invested: 0,
            active: false,
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

    const toggleNotes = (task) => {
        task.showNotes = !task.showNotes
    }

    const addNote = (task) => {
        task.notes.push({
            id: noteId++,
            text: task.note,
        })
        task.note = ""
    }

    const removeNote = (task, note) => {
        task.notes = task.notes.filter(n => n != note)
    }

    const exportData = () => {
        let data = {
            tasks: tasks.value,
        }
        const dataString = JSON.stringify(data, null, 2)
        navigator.clipboard.writeText(dataString)
        addAlert("Copied!")
    }

    const addAlert = (text) => {
        alerts.value.push({
            value: text,
            id: alertId++,
        })
        setTimeout(() => {
            alerts.value.shift()
        }, 2000)
    }

    onMounted(() => {
        nameref.value.focus()
    })

</script>

<template>

    <div class="alerts">
        <TransitionGroup tag="div" name="fade" class="container">
            <div v-for="alert in alerts" class="alert" :key="alert.id">
                {{ alert.value }}
            </div>
        </TransitionGroup>
    </div>

    <p>
        <a tabindex="-1" href="https://github.com/neyhden/tasks">
            <img class="upper-icon" src="./assets/github.svg" alt="GitHub">
        </a>
        <button tabindex="-1" @click="exportData">
            <img class="upper-icon" src="./assets/download.svg" alt="Export">
        </button>
    </p>

    <TransitionGroup tag="div" name="fade" class="container">
        <div v-for="task in tasks" :key="task.id">
            <div class="task">
                <button tabindex="-1" @click="clickedTask(task)">
                    <span>{{ task.name }}</span>
                    <span :class="task.active ? 'running' : 'stopped'">
                        {{ task.active ? "- Running" : "- Stopped" }}
                    </span>
                </button>
                <div>
                    <button @click="toggleNotes(task)" tabindex="-1">
                        <span>NOTES </span>
                        <img class="bin-img" src="./assets/note.svg" alt="[]">
                    </button>
                </div>
                <div>
                    <span :class="task.invested > task.effort ? 'overtime' : ''">
                        {{ secToFormat(task.invested) }}
                    </span>
                    <span class="time-divider">/</span>
                    <span :class="task.invested > task.effort ? 'overtime' : ''">
                        {{ secToFormat(task.effort) }}
                    </span>
                    <button tabindex="-1" class="bin" @click="removeTask(task)">
                        <span>DELETE </span>
                        <img class="bin-img" src="./assets/bin.svg" alt="X">
                    </button>
                </div>
            </div>
            <div v-if="task.showNotes" class="notes">
                <ul>
                    <li v-for="note in task.notes">
                        <button tabindex="-1" class="bin" @click="removeNote(task, note)">
                            <img class="bin-img" src="./assets/bin.svg" alt="X">
                        </button>
                        <span>
                            {{ note.text }}
                        </span>
                    </li>
                    <hr v-if="task.notes.length !== 0">
                    <textarea @keyup.enter="addNote(task)" placeholder="Add note" v-model="task.note" />
                </ul>
            </div>
        </div>

        <div class="task">
            <div>
                <input ref="nameref" @keyup.enter="addTask" v-model="taskName" placeholder="New task name">
            </div>
            <div>
                <span>Estimated effort: </span>
                <input size="1" maxlength="2" class="time-input"
                    @keyup.enter="addTask" v-model="taskEffortHH" placeholder="HH">
                <span> : </span>
                <input size="1" maxlength="2" class="time-input"
                    @keyup.enter="addTask" v-model="taskEffortMM" placeholder="MM">
                <span> : </span>
                <input size="1" maxlength="2" class="time-input"
                    @keyup.enter="addTask" v-model="taskEffortSS" placeholder="SS">
                <button tabindex="-1" @click="addTask" class="add-button">+</button>
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
        transition-duration: 0.2s;
        color: white;
        font-size: unset;
        user-select: none;
    }
    button:hover {
        background: rgba(255, 255, 255, 0.1);
    }
    button:focus {
        border: none;
    }

    input {
        background: rgba(255, 255, 255, 0.1);
        border: none;
        outline: none;
        box-shadow: none;
        color: white;
        border-radius: 100px;
        font-size: 18px;
        padding: 0px 20px 0px 20px;
        text-align: center;
    }

    textarea {
        background: rgba(255, 255, 255, 0.1);
        border: none;
        outline: none;
        box-shadow: none;
        color: white;
        border-radius: 10px;
        font-size: 18px;
        padding: 0px 20px 0px 20px;
        width: 90%;
    }

    time-input {
        width: 50px;
    }

    .alerts {
        position: absolute;
        left: 40%;
        right: 40%;
        display: flex;
        flex-direction: column;
    }
    .alert {
        text-align: center;
        font-weight: bolder;
        background: #ffaa77;
        border-radius: 100px;
        color: black;
        margin: 5px 0px 5px 0px;
    }

    .task {
        background: rgba(255, 255, 255, 0.1);
        transition-duration: 100ms;
        border-radius: 10px;
        margin: 5px 0px 5px 0px;
        padding: 2px 20px 2px 20px;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
    .task:hover {
        background: rgba(255, 255, 255, 0.15);
    }

    .notes {
        background: rgba(255, 255, 255, 0.10);
        border-radius: 0px 0px 15px 15px;
        padding: 10px;
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
        top: 3px;
        color: #ff2020;
        height: 20px;
    }

    .upper-icon {
        height: 40px;
    }

    /* Thank you vue examples */
    /* 1. declare transition */
    .fade-move,
    .fade-enter-active,
    .fade-leave-active {
      transition: all 0.3s cubic-bezier(0.55, 0, 0.1, 1);
    }

    /* 2. declare enter from and leave to state */
    .fade-enter-from {
      opacity: 0;
      transform: translate(0, 30px);
    }
    .fade-leave-to {
        opacity: 0;
        transform: translate(0, -30px);

    }

    /* 3. ensure leaving items are taken out of layout flow so that moving
          animations can be calculated correctly. */
    .fade-leave-active {
        position: absolute;
        right: 10px;
        left: 10px;
    }

</style>
