<script>
  import { initializeApp, getApps, getApp } from 'firebase/app'
  import {
    getFirestore,
    collection,
    onSnapshot,
    doc,
    updateDoc,
    deleteDoc,
    addDoc,
  } from 'firebase/firestore'
  import { firebaseConfig } from '$lib/firebaseConfig'
  import { browser } from '$app/environment'

  const firebaseApp =
    browser &&
    (getApps().length === 0 ? initializeApp(firebaseConfig) : getApp())

  const db = browser && getFirestore()

  const colRef = browser && collection(db, 'todos')

  let todos = []

  const unsubscribe =
    browser &&
    onSnapshot(colRef, (querySnapshot) => {
      let fbTodos = []
      querySnapshot.forEach((doc) => {
        let todo = { ...doc.data(), id: doc.id }
        fbTodos = [todo, ...fbTodos]
      })
      todos = fbTodos
    })
  console.log({ firebaseApp, db })

  let task = ''
  let error = ''
  const addTodo = async () => {
    if (task !== '') {
      const docRef = await addDoc(collection(db, 'todos'), {
        task: task,
        isComplete: false,
      })
      error = ''
    } else {
      error = 'Task is empty'
    }
    task = ''
  }

  const markTodoAsComplete = async (todo) => {
    await updateDoc(doc(db, 'todos', todo.id), {
      isComplete: !todo.isComplete,
    })
  }
  const deleteTodo = async (todo) => {
    await deleteDoc(doc(db, 'todos', todo.id))
  }
  const keyIsPressed = (e) => {
    if (e.key === 'Enter') addTodo()
  }
</script>

<input type="text" bind:value={task} />
<button on:click={addTodo}>Add</button>

<ol>
  {#each todos as todo}
    <li class:complete={todo.isComplete}>
      <span>
        {todo.task}
      </span>
      <span>
        <button on:click={() => markTodoAsComplete(todo)}>✓</button>
        <button on:click={() => deleteTodo(todo)}>✗</button>
      </span>
    </li>
  {:else}
    <p>No Tasks Found</p>
  {/each}
  <p class="error">{error}</p>
</ol>

<svelte:window on:keydown={keyIsPressed} />

<style>
  .complete {
    text-decoration: line-through;
  }
  .error {
    color: red;
  }
</style>
