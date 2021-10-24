<template>
    <div v-show="showAddTask">
        <AddTask @add-task="addTask" />
    </div>
    <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks"/>
</template>

<script>
import AddTask from '../components/AddTask'
import Tasks from '../components/Tasks'

export default {
    name: 'Home',
    props: {
        showAddTask: Boolean,
    },
    components: {
        AddTask,
        Tasks
    },
    data() {
        return {
            tasks: [],
        }
    },
    methods: {
        async deleteTask(id) {
            if (! confirm('Are you sure about that?')) {
                return;
            }

            const res = await fetch(`/api/tasks/${id}`, {method: 'DELETE'});

            if (res.status !== 200) {
                alert('Error deleting task');
                return;
            }

            this.tasks = this.tasks.filter((task) => task.id !== id);
        },
        async toggleReminder(id) {
            const task = await this.fetchTask(id);
            const toggledTask = {...task, reminder: !task.reminder};

            const res = await fetch(`/api/tasks/${id}`, {
                method: 'PUT',
                headers: {
                    'Content-type': 'application/json',
                },
                body: JSON.stringify(toggledTask)
            });

            const data = await res.json();

            this.tasks = this.tasks.map((task) => task.id === id ? {...task, reminder: data.reminder} : task);

            // How I did before seeing the video
            // const task = await this.fetchTask(id);

            // const res = await fetch(`/api/tasks/${id}`, {
            //     method: 'PATCH',
            //     headers: {
            //         'Content-type': 'application/json',
            //     },
            //     body: JSON.stringify({reminder: !task.reminder})
            // });

            // this.tasks = this.tasks.map((task) => task.id === id ? {...task, reminder: !task.reminder} : task);
        },
        async addTask(task) {
            const res = await fetch('/api/tasks', {
                method: 'POST',
                headers: {
                    'Content-type': 'application/json',
                },
                body: JSON.stringify(task)
            });

            const data = await res.json();

            this.tasks = [...this.tasks, data];

            // The way I would do
            // this.tasks.push(data);
        },
        async fetchTasks() {
            const res = await fetch('api/tasks');

            const data = await res.json();

            return data;
        },
        async fetchTask(id) {
            const res = await fetch(`api/tasks/${id}`);

            const data = await res.json();

            return data;
        }
    },
    async created() {
        this.tasks = await this.fetchTasks();
    }
}
</script>