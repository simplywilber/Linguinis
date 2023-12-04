<script>
  import TodoItem from "../../components/TodoItem.svelte";
  import { db } from "../../lib/firebase/firebase";
  import { authHandlers, authStore } from "../../store/store";
  import { getDoc, doc, setDoc } from "firebase/firestore";

  let todoList = [];

  let currTodo = "";
  let error = false;

  authStore.subscribe((curr) => {
    todoList = curr.data.todos;
  });

  // This allows adding or pushing into the todo list array by spreading, at the end the currTodo is reset
  function addTodo() {
    error = false;
    if (!currTodo) {
      error = true;
    }
    todoList = [...todoList, currTodo];
    currTodo = "";
  }
  //   CRUD functionality/logic
  function editTodo(index) {
    let newTodoList = todoList.filter((val, i) => {
      return i !== index;
    });
    currTodo = todoList[index];
    todoList = newTodoList;
  }

  function removeTodo(index) {
    let newTodoList = todoList.filter((val, i) => {
      return i !== index;
    });
    todoList = newTodoList;
  }
  async function saveTodos() {
    try {
      const userRef = doc(db, "users", $authStore.user.uid);
      await setDoc(
        userRef,
        {
          todos: todoList,
        },
        { merge: true }
      );
    } catch (error) {
      console.log("There was an error saving your information");
    }
  }
</script>

{#if !$authStore.loading}
  <div class="mainContainer">
    <div class="headerContainer">
      <h1>Todo List</h1>
      <div class="headerBtns">
        <button on:click={saveTodos}
          ><i class="fa-regular fa-floppy-disk" />
          <p>Save</p></button
        >
        <button on:click={authHandlers.logout}
          ><i class="fa-solid fa-right-from-bracket" />
          <p>Logout</p></button
        >
      </div>
    </div>
    <main>
      {#if todoList.length === 0}
        <p>You're all set!</p>
      {/if}
      {#each todoList as todo, index}
        <TodoItem {todo} {index} {removeTodo} {editTodo} />
      {/each}
    </main>
    <div class={"entries" + (error ? "errorBorder" : "")}>
      <input bind:value={currTodo} type="text" placeholder="Enter todo" />
      <button on:click={addTodo}>Add</button>
    </div>
  </div>
{/if}

<style>
  .mainContainer {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    gap: 24px;
    padding: 24px;
    width: 100%;
    max-width: 1000px;
    margin: 0 auto;
  }

  .headerContainer {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .headerBtns {
    display: flex;
    align-items: center;
    gap: 14px;
  }
  .headerContainer button {
    background: #546a4d;
    color: white;
    padding: 10px 18px;
    border: none;
    border-radius: 4px;
    font-weight: 700;
    display: flex;
    align-items: center;
    gap: 10px;
    cursor: pointer;
  }

  .headerContainer button i {
    font-size: 1rem;
  }

  .headerContainer button:hover {
    opacity: 0.7;
  }
  main {
    display: flex;
    flex-direction: column;
    gap: 8px;
    flex: 1;
  }

  .entries {
    display: flex;
    align-items: stretch;
    border: 1px solid #9cc88f;
    border-radius: 5px;
    overflow: hidden;
  }
  .errorBorder {
    border-color: coral !important;
  }
  .entries input {
    background: transparent;
    border: none;
    padding: 14px;
    color: white;
    flex: 1;
  }
  .entries input:focus {
    outline: none;
  }
  .entries button {
    padding: 0 28px;
    background: #546a4d;
    color: white;
    border: none;
    font-weight: 600;
    cursor: pointer;
  }
  .entries button:hover {
    background: transparent;
  }
</style>
