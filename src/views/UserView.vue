<template>
  <div v-if="user">
    <nav-bar></nav-bar>

    <div class="container mt-4">
      <div class="card shadow-sm p-4 mb-4">
        <h2 class="card-title mb-3">Profile</h2>
        <p class="mb-2"><strong>Email:</strong> {{ user.email }}</p>
        <p class="mb-2"><strong>Name:</strong> {{ displayName || "Not set" }}</p>
        <img v-if="user.photoURL" :src="user.photoURL" alt="Profile Picture" class="img-thumbnail" width="120" />
      </div>

      <div class="card shadow-sm p-4">
        <h3 class="mb-3">Change Password</h3>

        <div class="mb-3">
          <label for="currentPassword" class="form-label">Current Password</label>
          <input type="password" id="currentPassword" v-model="currentPassword" class="form-control" />
        </div>

        <div class="mb-3">
          <label for="newPassword" class="form-label">New Password</label>
          <input type="password" id="newPassword" v-model="newPassword" class="form-control" />
        </div>

        <div class="mb-3">
          <label for="confirmPassword" class="form-label">Confirm Password</label>
          <input type="password" id="confirmPassword" v-model="confirmPassword" class="form-control" />
        </div>

        <button class="btn btn-primary" @click="updatePassword">Confirm</button>
      </div>

      <div class="card shadow-sm p-4 mt-4">
        <h3 class="mb-3">Change User Name</h3>
        <div class="mb-3">
          <label for="newUsername" class="form-label">New user name</label>
          <input id="newUsername" v-model="newUserName" class="form-control" />
          <p class="text-danger">{{usernameError}}</p>
        </div>
          <button class="btn btn-primary" @click="updateUserName">Confirm</button>
      </div>

    </div>
  </div>
</template>
  
  <script setup>
  import NavBar from "@/components/NavBar.vue";
  import { ref, onMounted } from "vue";
  import { auth } from "@/firebase/config";
  import firebase from "firebase/app"; 
  import { db } from "@/firebase/config";
  import { useRouter } from "vue-router";
  
  
  const user = ref(null);
  const displayName = ref("");
  const currentPassword = ref("");
  const newPassword = ref("");
  const confirmPassword = ref("");
  const newUserName = ref("");
  const usernameError = ref("");
  const router = useRouter();
  
  onMounted(() => {
    auth.onAuthStateChanged(current => {
      if (current) {
        user.value = current;
        displayName.value = current.displayName;
      }
    });
  });
  
  const updatePassword = async () => {
    if (newPassword.value !== confirmPassword.value) {
      alert("Passwords do not match!");
      return;
    }
  
    try {
      const credential = firebase.auth.EmailAuthProvider.credential(
        user.value.email,
        currentPassword.value
      );
  
      // Re-authenticate the user
      await user.value.reauthenticateWithCredential(credential);
  
      // Now update the password
      await user.value.updatePassword(newPassword.value);
      alert("Password updated successfully!");
    } catch (error) {
      console.error("Error updating password:", error.message);
      alert("Failed to update password: " + error.message);
    }
  };

  const updateUserName = async () => {
    usernameError.value = "";

    let takens = ref([]);
    await db.collection("users").get().then((snapshot) => {
      snapshot.forEach((doc) => {
        takens.value.push(doc.data().name);
      })
    });

    if (newUserName.value === "") {
      usernameError.value = "Username cannot be empty!";
      return;
    }
    else if (takens.value.includes(newUserName.value)) {
      usernameError.value = "Username already taken!";
      return;
    }
    else if (newUserName.value.length < 3) {
      usernameError.value = "Username must be at least 3 characters long!";
      return;
    }
    else if (newUserName.value.length > 20) {
      usernameError.value = "Username must be at most 20 characters long!";
      return;
    }
    else if (!/^[a-zA-Z0-9]+$/.test(newUserName.value)) {
      usernameError.value = "Username can only contain letters and numbers!";
      return;
    }
  
    else{
      
      try{
        
        const userRef = db.collection("users").doc(user.value.uid);
        await userRef.update({
          name: newUserName.value
        })
        
        await auth.currentUser.updateProfile({
          displayName: newUserName.value
        })
       displayName.value = newUserName.value;
        alert("Username updated successfully!");
        newUserName.value = "";
        router.push('/profile')
      }
      catch (error) {
        console.error("Error updating username:", error.message);
        alert("Failed to update username: " + error.message);
      }
      

    } 
  }
  </script>
