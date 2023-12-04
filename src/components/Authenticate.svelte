<script>
  import { auth } from "../lib/firebase/firebase";
  import { authHandlers } from "../store/store";

  // This is like sveltes version for handling events and changes (amazing, i know)
  let email = "";
  let password = "";
  let confirmPass = "";
  let error = false;
  let register = false;
  let authenticating = false;

  async function handleAuthenticate() {
    if (authenticating) {
      return;
    }
    if (!email || !password || (register && !confirmPass)) {
      error = true;
      return;
    }
    authenticating = true;

    try {
      if (!register) {
        await authHandlers.login(email, password);
      } else {
        await authHandlers.signup(email, password);
      }
    } catch (err) {
      console.log("There was an authentication error", err);
      error = true;
      authenticating = false;
    }
  }

  function handleRegister() {
    register = !register;
  }
</script>

<!-- Binding helps with the event handling configuration -->

<div class="authContainer">
  <form>
    <h1>{register ? "Register" : "Login"}</h1>

    <!-- Conditional statment which renders an error if the condition is true -->
    {#if error}
      <p class="error">The information you have entered is not correct</p>
    {/if}
    <label>
      <p class={email ? "above" : "center"}>Email</p>
      <input bind:value={email} type="email" placeholder="Email" />
    </label>
    <label>
      <p class={password ? "above" : "center"}>Password</p>
      <input bind:value={password} type="password" placeholder="Password" />
    </label>
    <!-- This will render when register is true -->
    {#if register}
      <label>
        <p class={confirmPass ? "above" : "center"}>Confirm Password</p>
        <input
          bind:value={confirmPass}
          type="password"
          placeholder="Confirm Password"
        />
      </label>
    {/if}
    <button on:click={handleAuthenticate} type="button" class="submitBtn">
      {#if authenticating}
        <i class="fa-solid fa-circle-notch spin" />
      {:else}
        Submit
      {/if}
    </button>
  </form>

  <!-- This provides the user the option to navigate the authentication page -->
  <div class="options">
    <p>or</p>
    {#if register}
      <div>
        <p>Already have an account?</p>
        <!-- allows the text to be clickable and change the state-->
        <p on:click={handleRegister} on:keydown={() => {}}>Login</p>
      </div>
    {:else}
      <div>
        <p>Don't have an account?</p>
        <p on:click={handleRegister} on:keydown={() => {}}>Register</p>
      </div>
    {/if}
  </div>
</div>

<style>
  .authContainer {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    /* This allows the container to occupy the full parent width */
    flex: 1;
    padding: 24px;
  }

  form {
    display: flex;
    flex-direction: column;
    gap: 14px;
    width: 400px;
    max-width: 100%;
    margin: 0 auto;
  }

  form,
  .options {
    width: 400px;
    max-width: 100%;
    margin: 0 auto;
  }

  form input {
    width: 100%;
  }
  h1 {
    text-align: center;
    font-size: 3rem;
  }

  form label {
    position: relative;
    border: 1px solid #7e8d7a;
    border-radius: 5px;
  }

  /* This styles the border around the inputs */
  form input {
    border: none;
    background: transparent;
    color: white;
    padding: 14px;
  }

  form input:focus {
    border: none;
    outline: none;
  }

  form label:focus-within {
    border-color: #dfe5db;
  }
  form button {
    background: #9dc08b;
    color: rgb(246, 243, 243);
    border: none;
    padding: 14px 0;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1rem;
    display: grid;
    place-items: center;
  }
  form button:hover {
    background: #afd89c;
  }
  .spin {
    animation: spin 1.5s linear infinite;
  }
  @keyframes spin {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }
  /* This styles the little hover elements above the inputs */
  .above,
  .center {
    position: absolute;
    transform: translateY(-50%);
    pointer-events: none;
    color: white;
    border-radius: 4px;
    padding: 0 6px;
    font-size: 0.8rem;
  }

  .above {
    top: 0;
    left: 24px;
    background: #40513b;
    border: 3px solid #9dc08b;
    font-size: 0.7rem;
  }

  .center {
    top: 50%;
    left: 6px;
    border: 1px solid transparent;
    opacity: 0;
  }

  .error {
    color: coral;
    font-size: 0.9rem;
    text-align: center;
  }

  /* styles the options, also includes line styling */
  .options {
    padding: 14px 0;
    overflow: hidden;
    font-size: 0.9rem;
    display: flex;
    flex-direction: column;
    gap: 4px;
  }
  .options > p {
    position: relative;
    text-align: center;
    width: fit-content;
    margin: 0 auto;
    padding: 0 8px;
  }

  .options > p::after,
  .options > p::before {
    position: absolute;
    content: "";
    top: 50%;
    transform: translateY(-50%);
    width: 100vw;
    height: 1.5px;
    background: white;
  }

  .options > p::after {
    right: 100%;
  }
  .options > p::before {
    left: 100%;
  }

  .options div {
    display: flex;
    align-items: center;
    gap: 8px;
    justify-content: center;
  }
  .options div p:last-of-type {
    color: #90ff59;
    cursor: pointer;
    justify-content: center;
  }
</style>
