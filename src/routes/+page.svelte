<script lang="ts">
  import {getToastStore} from "@skeletonlabs/skeleton"
  import type {ToastSettings} from "@skeletonlabs/skeleton"
  import { setDoc, doc, getDoc, onSnapshot, collection } from "firebase/firestore"
  import {db, auth} from "./../lib/db"
  import { onAuthStateChanged } from "firebase/auth";
  import { Circle } from "svelte-loading-spinners";

  let user: any = '2314124+4141425+8467+120'
  let displayName: string | null = ''

onAuthStateChanged(auth, (currentUser) => {
  if (currentUser) {
    user = currentUser
    if (currentUser.displayName) {
      displayName = currentUser.displayName
    } else {
      displayName = currentUser.email
    }
    checkIfRatHasBeenSentWithinLastTenMinutes()
  } else {
    user = ''
  }
});


  const toastStore = getToastStore()

const toastMessage = (message: string) => {
    const toast: ToastSettings = {
    message: message,
    hideDismiss: true,
    background: "border border-secondary-500"
    }
    toastStore.trigger(toast)
}

let ratsSent: number

let disabled = false

const sendRat = async () => {
  if (hasRatBeenSentWithinLastTenMinutes) {
    disabled = true
  }
  else {
    try {
      await setDoc(doc(db, "rats", user.uid), {
        sentAt: new Date().getTime(),
        displayName: displayName,
        ratsSent: ratsSent += 1
      })
      if (intervalId) {
        clearInterval(intervalId)
      }
      checkIfRatHasBeenSentWithinLastTenMinutes()
      disabled = true
    } catch (err) {
      disabled = false
      console.log(err)
    }
  }
}

  let loaded = false

  let pending = false

  let intervalId: any;

  let hasRatBeenSentWithinLastTenMinutes = false
  let canRatIn: any
  const checkIfRatHasBeenSentWithinLastTenMinutes = async () => {
    try {
      const docRef = doc(db, "rats", user.uid);
      const docSnap = await getDoc(docRef);

      if (docSnap.exists()) {
        ratsSent = docSnap.data().ratsSent
        const sentAt = docSnap.data().sentAt;
        const oneMinuteAgo = new Date().getTime() - 1 * 60 * 1000;
        canRatIn = sentAt - oneMinuteAgo
        loaded = true

        if (sentAt < oneMinuteAgo) {
          hasRatBeenSentWithinLastTenMinutes = false
        } else {
          hasRatBeenSentWithinLastTenMinutes = true
          disabled = true
          updateCountdown(); 
          intervalId = setInterval(updateCountdown, 1000);
          pending = false
        }
      } else {
        loaded = true
        ratsSent = 0
      }
    } catch (error) {
      console.log(error);
    }
  }
const updateCountdown = () => {
    if (canRatIn !== null) {
      canRatIn -= 1000; 
    }
  if (canRatIn <= 0) {
    clearInterval(intervalId)
    disabled = false
    checkIfRatHasBeenSentWithinLastTenMinutes()
  }
  };

  const query = collection(db, "rats")
  onSnapshot(query, (snapshot) => {
    snapshot.docChanges().forEach((change) => {
      if (change.type === "modified") {
        toastMessage(`${change.doc.data().displayName} just ratted`)
      }
    })
  })



</script>

<main class="absolute inset-0 flex flex-col justify-center items-center gap-12 font-bold text-2xl">
  <img src="https://i.ibb.co/yPQ4fbG/xdd.png" alt="xdd" class="w-80" />
  {#if user}
  {#if loaded}
    <button 
      disabled={disabled} 
      on:click={() => {pending = true, disabled = true, sendRat()}} 
      class="rounded-md border border-secondary-500 w-80 p-3">
        {#if disabled}
          {#if pending}
            <div class="flex gap-20"><Circle color={"#3B81F6"} size={32} />Pending</div>
          {:else}
            {`You can rat in ${Math.floor(canRatIn / 1000)} sec`}
          {/if}
        {:else}
          Send your rat
        {/if}
    </button>  
  {:else}
  <button 
    disabled 
    class="rounded-md border border-secondary-500 w-80 p-3 flex items-center gap-20"><Circle color={"#3B81F6"} size={32} />Loading</button>
  {/if}
  {:else}
  <button disabled class="rounded-md border border-secondary-500 w-80 p-3">You must log in to rat</button>
  {/if}
</main>
