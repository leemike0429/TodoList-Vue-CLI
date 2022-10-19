<template>
    <div class="container-sm p-0" id="TodoList">
        <div class="col-12">
            <div class="input-row mb-3">
                <span class="col-2 text-nowrap ">待辦事項</span>
                <input class="col-9" type="text" v-model="todoInput" placeholder="準備想做的任務" />
                <button class="btn col-1"  @keyup.enter.once="addTodo(todoInput)"
                    @click="addTodo(todoInput)">新增</button>
            </div>
            <ul class="nav nav-tabs w-100 " id="myTab" role="tablist">
                <li class="nav-item " role="presentation">
                    <button class="nav-link active " id="home-tab" data-bs-toggle="tab" data-bs-target="#home-tab-pane"
                        type="button" role="tab" aria-controls="home-tab-pane" aria-selected="true">全部</button>
                </li>
                <li class="nav-item" role="presentation">
                    <button class="nav-link" id="profile-tab" data-bs-toggle="tab" data-bs-target="#profile-tab-pane"
                        type="button" role="tab" aria-controls="profile-tab-pane" aria-selected="false">進行中</button>
                </li>
                <li class="nav-item" role="presentation">
                    <button class="nav-link" id="contact-tab" data-bs-toggle="tab" data-bs-target="#contact-tab-pane"
                        type="button" role="tab" aria-controls="contact-tab-pane" aria-selected="false">已完成</button>
                </li>
            </ul>
            <div class="tab-content" id="myTabContent">
                <div class="tab-pane fade show active" id="home-tab-pane" role="tabpanel" aria-labelledby="home-tab"
                    tabindex="0">
                    <template v-for="item in todoList.all" :key="item">
                        <inputRow :row-Value="item" @remove="remove"></inputRow>
                    </template>
                </div>
                <div class="tab-pane fade" id="profile-tab-pane" role="tabpanel" aria-labelledby="profile-tab"
                    tabindex="0">
                    <template v-for="item in todoList.on" :key="item">
                        <inputRow :row-Value="item" @remove="remove"></inputRow>
                    </template>
                </div>
                <div class="tab-pane fade" id="contact-tab-pane" role="tabpanel" aria-labelledby="contact-tab"
                    tabindex="0">
                    <template v-for="item in todoList.done" :key="item">
                        <inputRow :row-Value="item" @remove="remove"></inputRow>
                    </template>
                </div>
                <div class="tab-content-footer">
                    <span>還有 {{todoList.on.length}} 筆任務未完成</span>
                    <span class="clickable" @click="cleanAllTodo()">清除所有任務</span>
                </div>
            </div>
        </div>
    </div>
</template>
<style lang="scss" scoped>
//plugins
@import "bootstrap/dist/css/bootstrap.min.css";
@import "../scss/_variable.scss";
@import "../scss/_base.scss";
#TodoList {
    border: 1px solid $border;
    width: 700px;
    border-radius: 0 0.25rem 0 0;
    background-color:$light-gray;
    .input-row {
        border-bottom: 1px solid $border;
        display: flex;
        align-items: center;
        background-color: $main-gray;

        span {
            display: block;
            padding: 0.3rem 0.75rem;
            height: 100%;
            border-right: 1px solid $border;
            color:$darker-gray;
            font-weight: 600;
            letter-spacing: 1px;
        }

        input {
            border: none;
            padding: 0.375rem 0.75rem;
        }

        button {
            border-radius: 0 0.25rem 0 0;
            color:#000;
            background-color: $main-green; 
        }

    }

    ul {
        background-color: $main-gray;
        border-top: 1px solid $border;
        padding-top: 1rem;
        li {
            white-space: nowrap;

            &:first-child {
                margin-left: 1rem;
            }

            button {
                color: $main-green;
            }

            .active {
                color:$darker-gray;
            }
        }
    }

    .tab-content {
        max-height: 500px;
        overflow-y: auto;
        background-color: $main-gray;
        display: flex;
        flex-direction: column;

        &-footer {
            margin-top: auto;
            padding: 1rem;
            display: flex;
            justify-content: space-between;

            span {
                color:$darker-gray;
                &:last-child {
                    color: $main-green;
                }
            }
        }
    }
}
</style>
<script>
import inputRow from '../components/TodoRow.vue';
import helper from '../helper/helper.js'
import { computed, ref, reactive } from 'vue';
export default {
    components: {
        inputRow: inputRow
    },
    setup() {
        //#region data
        let todoInput = ref('');
        let allTodo = reactive({ list: [] });
        let todoList = reactive({
            all: [],
            on: [],
            done: [],
        });
        //#endregion

        //#region method
        //重製LocalStorage
        const reBuildLocalStorageValue = function () {
            localStorage.removeItem('todo');
            localStorage.setItem('todo', JSON.stringify(allTodo.list));
        }
        //移除一筆待辦事項
        const remove = function (value) {
            allTodo.list = reactive(allTodo.list.filter(x => x != value));
            reBuildLocalStorageValue();
        }
        //清除所有Todo
        const cleanAllTodo = function () {
            localStorage.removeItem('todo');
            allTodo.list = allTodo.list.filter(x => x.id == "");
        }
        //清除input
        const resetTodoInput = function () {
            todoInput.value = '';
        }
        //新增一筆待辦事項
        const addTodo = function (text) {
            if(text===''){
                alert("請輸入任務");
                return;
            }
            const guid = helper.generateGuid();
            const todoItem = {
                id: guid,
                info: text,
                isDone: false,
            };
            allTodo.list.push(todoItem);
            reBuildLocalStorageValue();
            resetTodoInput();
            getTodoFromLocalStorage();
        }
        //從localStorage拿資料
        const getTodoFromLocalStorage = function () {
            let tempValue = localStorage.getItem('todo');
            if (tempValue && tempValue.length > 0) {
                allTodo.list = JSON.parse(tempValue);
            }
        }
        //#endregion

        //#region computed
        todoList.all = computed(() =>
            allTodo.list
        );

        todoList.on = computed(() => {
            return allTodo.list.filter(x => !x.isDone)
        });

        todoList.done = computed(() => {
            return allTodo.list.filter(x => x.isDone)
        });
        //#endregion

        //init Data
        getTodoFromLocalStorage();

        return { todoList, todoInput, remove, cleanAllTodo, resetTodoInput, addTodo };
    },
}
</script>
