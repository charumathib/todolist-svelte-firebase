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

  // initializes connection to firebase backend and gets our "todos" database
  browser && (getApps().length == 0 ? initializeApp(firebaseConfig) : getApp());
  const db = browser && getFirestore();
  const colRef = browser && collection(db, "todos");

  // list to store all todos which are objects with the following fields
  // { 
  //   task: "task name",
  //   completed: true if task is completed, otherwise false
  //   createdAt: timestamp at which the task was created 
  // }
  let todos = [];
  let task = "";

  // queries all objects from "todos" database
  browser &&
    onSnapshot(colRef, (querySnapshot) => {
      let fbTodos = [];
      querySnapshot.forEach((doc) => {
        // extracts fields from todo object (i'm unsure if there is a better way to do this)
        let todo = { ...doc._document.data.value.mapValue.fields, id: doc.id };
        fbTodos = [todo, ...fbTodos];
      });
      // locally store all of the todos
      todos = fbTodos;
    });

  // adds a todo
  const addTodo = async () => {
    // if a task name has been written in the input box...
    if (task != "") {
      // add the todo object to firebase
      await addDoc(collection(db, "todos"), {
        task: task,
        completed: false,
        createdAt: new Date(),
      });
    }
    // reset the task variable so there is no text in the input box
    task = "";
  };

  // marks a todo as completed
  const markTodoDone = async (id) => {
    // update the todo object (by id) in firebase to have the completed field set to true
    await updateDoc(doc(db, "todos", id), {
      completed: true,
    });
  };

  // deletes a todo from firebase (by id)
  const deleteTodo = async (id) => {
    await deleteDoc(doc(db, "todos", id));
  };
</script>

<!-- input box and add button -->
<div class="inputs">
  <!-- binding the input value to "task" populates the content of the "task" variable with
       whatever has been typed in the input box after every keystroke  -->
  <input type="text" placeholder="Add a task" bind:value={task} />
  <button class="addButton" on:click={addTodo}> Add </button>
</div>

<div class="container">
  <!-- loop through each todo in the "todo"s list -->
  {#each todos as todo}
    <!-- change formatting based on whether or not a todo has been completed -->
    <div
      class="task"
      class:incomplete={!todo.completed.booleanValue}
      class:complete={todo.completed.booleanValue}
    >
      <span class="todoVal">
        {todo.task.stringValue}
      </span>
      <!-- check and x buttons for completing and deleting todos -->
      <span>
        <button on:click={() => markTodoDone(todo.id)}>✓</button>
        <button on:click={() => deleteTodo(todo.id)}>✗</button>
      </span>
    </div>
  {/each}
</div>

<style>
  /* styling for input box */
  input {
    width: 50%;
    padding: 12px 20px;
    margin: 8px 0;
    box-sizing: border-box;
    font-size: 30px;
  }

  /* styling of task background for when a task is complete or incomplete */
  .complete {
    background-color: darkgray;
  }
  .incomplete {
    background-color: peachpuff;
  }
  /* styles the containers that holds the todos so that the tasks are vertically stacked */
  .container {
    position: relative;
    width: 50%;
    height: 100%;
    display: flex;
    text-align: center;
    flex-direction: column;
  }

  /* styles the content of the task div */
  .task {
    padding-left: 100px;
    padding-right: 100px;
    display: flex;
    justify-content: space-between;
    margin: 10px;
  }

  /* styles the text of the task */
  .todoVal {
    padding-top: 15px;
  }

  /* styles the "Add" button */
  .addButton {
    border: 5px solid peachpuff;
    margin: 5px;
  }

  /* styles the check and x buttons */
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

  /* styles all spans (essentially just the elements within the todo div) */
  span {
    font-size: 30px;
  }
</style>
