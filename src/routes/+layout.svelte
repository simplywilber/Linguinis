<script>
  //This is on page load
  import { onMount } from "svelte";
  import { auth, db } from "../lib/firebase/firebase";
  import { getDoc, doc, setDoc } from "firebase/firestore";
  import { authStore } from "../store/store";

  const nonAuthRoutes = ["/"];

  onMount(() => {
    console.log("Mounting");

    //onAuthStateChanged listens to login logout user registration
    const unsubscribe = auth.onAuthStateChanged(async (user) => {
      const currentPath = window.location.pathname;

      //prevents non-authenticated users from accessing authenticated pages
      if (!user && !nonAuthRoutes.includes(currentPath)) {
        window.location.href = "/";
        return;
      }

      if (user && currentPath == "/") {
        window.location.href = "/dashboard";
        return;
      }

      if (!user) {
        return;
      }
      //A firebase method where the user data can be accessed
      let dataToSetToStore;

      const docRef = doc(db, "users", user.uid);
      const docSnap = await getDoc(docRef);
      if (!docSnap.exists()) {
        const userRef = doc(db, "users", user.uid);
        dataToSetToStore = {
          email: user.email,
          todos: [],
        };

        await setDoc(
          userRef,
          dataToSetToStore,
          //Merges existing information istead of overriding it
          { merge: true }
        );
      } else {
        const userData = docSnap.data();
        dataToSetToStore = userData;
      }
      authStore.update((curr) => {
        return {
          ...curr,
          user,
          data: dataToSetToStore,
          loading: false,
        };
      });
    });
  });
</script>

<div class="mainContainer">
  <slot />
</div>

<style>
  /* This styles the main login page */
  .mainContainer {
    min-height: 100vh;
    background: linear-gradient(to right, #40513b, #609966);
    color: white;
    position: relative;
    display: flex;
    flex-direction: column;
  }
</style>
