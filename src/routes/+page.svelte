<script>
  import { initializeApp, getApps, getApp } from "firebase/app";
  import {
    getFirestore,
    collection,
    onSnapshot,
    doc,
    addDoc,
    updateDoc,
    deleteDoc,
  } from "firebase/firestore";
  import { firebaseConfig } from "$lib/firebaseConfig";
  import { browser } from "$app/environment";

  browser && (getApps().length == 0 ? initializeApp(firebaseConfig) : getApp());
  const db = browser && getFirestore();

  const colRef = browser && collection(db, "todos");

  browser &&
    onSnapshot(colRef, (querySnapshot) => {
      let fbTodos = [];
      querySnapshot.forEach((doc) => {
        let todo = { ...doc._document.data.value.mapValue.fields, id: doc.id };
        fbTodos = [todo, ...fbTodos];
      });
      todos = fbTodos;
    });

  let todos = [];
  let task = "";

  const addTodo = async () => {
    if (task != "") {
      await addDoc(collection(db, "todos"), {
        task: task,
        completed: false,
        createdAt: new Date(),
      });
    }
    task = "";
  };

  const markTodoDone = async (id) => {
    await updateDoc(doc(db, "todos", id), {
      completed: true,
    });
  };

  const deleteTodo = async (id) => {
    await deleteDoc(doc(db, "todos", id));
  };
</script>

<div class="inputs">
  <input type="text" placeholder="Add a task" bind:value={task} />
  <button class="addButton" on:click={addTodo}> Add </button>
</div>

<div class="container">
  {#each todos as todo}
    <div class="task" class:incomplete={!todo.completed.booleanValue} class:complete={todo.completed.booleanValue}>
      <span class="todoVal">
        {todo.task.stringValue}
      </span>
      <span>
        <button on:click={() => markTodoDone(todo.id)}>✓</button>
        <button on:click={() => deleteTodo(todo.id)}>✗</button>
      </span>
    </div>
  {/each}
</div>

<style>
  input {
    width: 50%;
    padding: 12px 20px;
    margin: 8px 0;
    box-sizing: border-box;
    font-size:30px
  }

  .complete {
    background-color:darkgray;
  }
  .incomplete {
    background-color:peachpuff;
  }
  .container {
    position: relative;
    width: 50%;
    height: 100%;
    display: flex;
    text-align: center;
    flex-direction: column;
  }
  .task {
    padding-left: 100px;
    padding-right: 100px;
    display: flex;
    justify-content: space-between;
    margin: 10px;
  }
  .todoVal {
    padding-top: 15px;
  }
  .addButton {
    border: 5px solid peachpuff;
    margin: 5px
  }
  button {
    border: none;
    color: black;
    padding: 15px 32px;
    text-align: center;
    background-color: transparent;
    text-decoration: none;
    display: inline-block;
    font-size: 30px;
  }
  span {
    font-size: 30px;
  }
</style>
