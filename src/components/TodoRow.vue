<template>
    <div class="todo-row d-flex w-100" v-if="rowValue">
        <label class="clickable">
            <input type="checkbox" :checked="todo.isDone" v-model="todo.isDone">
            <span class="text" :class="{'done':todo.isDone}">{{todo.info}}({{getStatusStr(todo.isDone)}})</span>
        </label>
        <span class="clickable remove-icon" @click="removeTodoItem()">x</span>
    </div>
</template>
<style lang="scss" scoped>
//plugins
@import "bootstrap/dist/css/bootstrap.min.css";
@import "../scss/_variable.scss";
@import "../scss/_base.scss";

.todo-row {
    box-shadow: 0px 1px 1px $box-shadow;
    margin-bottom: 0.2rem;
    padding: 0.8rem 1rem;
    background-color: #FFF;

    label {
        span {
            color: $darker-gray;
        letter-spacing: 1px;
        }

        input {
            margin-right: 0.3rem;
        }

        .done {
            text-decoration: line-through;
        }
    }


    .remove-icon {
        font-size: 1.2rem;
        margin-left: auto;
    }
}
</style>
<script >
import { reactive } from 'vue';
export default {
    props: ['rowValue'],
    setup(props, context) {
        //data
        let todo = reactive(props.rowValue);

        //methods
        const removeTodoItem = () => {
            context.emit("remove", props.rowValue);
        };
        const getStatusStr = function (isDone) {
            return isDone ? "已完成" : "進行中";
        };
        const changeInputStatus = function () {
            todo.isDone = !this.todo.isDone;
        };
        return { todo, removeTodoItem, getStatusStr, changeInputStatus }
    },

}
</script>
