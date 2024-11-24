# Login Menggunakan Firebase

Nama : Rassya Hafizh Suharjo

NIM : H1D022068

SHIFT : A(Baru) D(Lama)

## Router
Berdasarkan router pada aplikasi ini, halaman home yang ditampilkan secara default merupakan halaman /login. Namun, halaman utama juga bisa berganti langsung ke /home jika variabel isAuth nya true. Jika false, maka akan diarahkan ke halaman login. Profile Page dapat diakses setelah masuk ke halaman home atau harus sudah login dengan isAuth true.

```ts
const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    redirect: '/login',
  },
  {
    path: '/login',
    name: 'login',
    component: LoginPage,
    meta: {
      isAuth: false,
    },
  },
  {
    path: '/home',
    name: 'home',
    component: HomePage,
    meta: {
      isAuth: true,
    },
  },
  {
    path: '/profile',
    name: 'profile',
    component: ProfilePage,
    meta: {
      isAuth: true,
    },
  },
];

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
});
```

```ts
router.beforeEach(async (to, from, next) => {
  const authStore = useAuthStore();

  if (authStore.user === null) {
    await new Promise<void>((resolve) => {
      const unsubscribe = onAuthStateChanged(auth, () => {
        resolve();
        unsubscribe();
      });
    });
  }

  if (to.path === '/login' && authStore.isAuth) {
    next('/home');
  } else if (to.meta.isAuth && !authStore.isAuth) {
    next('/login');
  } else {
    next();
  }
});
```

## Koneksi Firebase
Koneksi firebase terdapat di 'firebas.ts'. Pada 'firebase.ts' dilakukan inisialisasi koneksi ke firebase pada aplikasi dengan konfigurasi sesuai database dari akun firebase yang ingin dihubungkan.
```ts
import { initializeApp } from "firebase/app";
import { getAuth, GoogleAuthProvider } from 'firebase/auth';
const firebaseConfig = {
  apiKey: "AIzaSyCSAAM1QUpD6erFSDTpa9s-s5s_8njIc2c",
  authDomain: "vue-firebase-cff27.firebaseapp.com",
  projectId: "vue-firebase-cff27",
  storageBucket: "vue-firebase-cff27.firebasestorage.app",
  messagingSenderId: "117211972129",
  appId: "1:117211972129:web:bfd63a27dfcb2ad5575960"
};

// Initialize Firebase
const firebase = initializeApp(firebaseConfig);
const auth = getAuth(firebase);
const googleProvider = new GoogleAuthProvider();

export { auth, googleProvider };
```

## Halaman Login
Pada halaman 'LoginPage.ts' terdapat title dan button untuk melakukan login ke google melalui firebase authentication.
```html
<div id="container">
                <!-- Title -->
                <ion-text style="margin-bottom: 20px; text-align: center;">
                    <h1>Praktikum Pemrograman Mobile</h1>
                </ion-text>

                <!-- Button Sign In -->
                <ion-button @click="login" color="light">
                    <ion-icon slot="start" :icon="logoGoogle"></ion-icon>
                    <ion-label>Sign In with Google</ion-label>
                </ion-button>
            </div>
```

Terdapat variabel asynchronous yang menunggu user untuk mengklik button login untuk memanggil fungsi LoginWithGoogle() dari 'auth.ts' untuk dijalankan dengan useAuthStore yaitu store dari pinia.
```ts
import { useAuthStore } from '@/stores/auth';
const authStore = useAuthStore();

const login = async () => {
    await authStore.loginWithGoogle();
};
```

Pada halaman 'auth.ts' dilakukan penyimpanan data user ke firebase authentication. Jika data user berhasil tersimpan ke firebase, maka di firebase console pada tabel user di firebase authentication akan terisi email google yang tersambung ke pengguna. User.value kemudian tidak null sehingga isAuth menjadi true dan user akan diarahkan ke halaman home. Terdapat juga alert apabila login gagal dilakukan.
```ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import router from '@/router';
import { auth } from '@/utils/firebase';
import { GoogleAuthProvider, onAuthStateChanged, signInWithCredential, signOut, User } from 'firebase/auth';
import { GoogleAuth } from '@codetrix-studio/capacitor-google-auth';
import { alertController } from '@ionic/vue';
export const useAuthStore = defineStore('auth', () => {
    // Variabel User
    const user = ref<User | null>(null);

    // Sign In with Google
    const loginWithGoogle = async () => {
        try {
            await GoogleAuth.initialize({
                clientId: '117211972129-d19fcuq0ievvtpfkbf3buqtmh6jodu2f.apps.googleusercontent.com',
                scopes: ['profile', 'email'],
                grantOfflineAccess: true,
            });

            const googleUser = await GoogleAuth.signIn();

            const idToken = googleUser.authentication.idToken;

            const credential = GoogleAuthProvider.credential(idToken);

            const result = await signInWithCredential(auth, credential);

            user.value = result.user;

            router.push("/home");
        } catch (error) {
            console.error("Google sign-in error:", error);
            
            const alert = await alertController.create({
                header: 'Login Gagal!',
                message: 'Terjadi kesalahan saat login dengan Google. Coba lagi.',
                buttons: ['OK'],
            });

            await alert.present();

            throw error;
        }
    };
```

Penjelasan cara login :
```ts
const googleUser = await GoogleAuth.signIn();
```
Memanggil metode signIn() dari library Capacitor Google Auth untuk memulai proses login menggunakan akun Google.

```ts
const idToken = googleUser.authentication.idToken;
```
Mengambil ID Token dari data autentikasi yang disediakan oleh Google. ID Token adalah token berbasis JWT yang dihasilkan oleh Google setelah autentikasi berhasil. Token ini digunakan untuk mengidentifikasi pengguna secara unik dalam aplikasi.

```ts
const credential = GoogleAuthProvider.credential(idToken);
```
Membuat credential Firebase dari ID Token menggunakan metode GoogleAuthProvider.credential. Credential digunakan oleh Firebase Authentication untuk memvalidasi pengguna terhadap backend Firebase memungkinkan aplikasi untuk mengenali pengguna yang masuk sebagai pengguna yang valid di Firebase.

```ts
const result = await signInWithCredential(auth, credential);
```
Menggunakan credential untuk masuk ke Firebase Authentication melalui metode signInWithCredential.

```ts
user.value = result.user;
```
Menyimpan informasi pengguna yang berhasil login ke dalam variabel user.

Kemudian ketika user.value sudah terisi maka variabel isAuth menjadi true
```ts
const isAuth = computed(() => user.value !== null);
```

![Page Login](gambar/login1.png)
![Login Google](gambar/login2edit.jpg)



## Halaman Home
![Halaman Home](gambar/home.png)

Halaman dapat diakses ketika variabel isAuth dalam keadaan true. Umumnya terjadi setelah melakukan login atau ketika membuka kembali aplikasi sebelum logout. Pada halaman home terdapat dua tab menu yang diambil dari 'TabsMenu.vue'. Terdapat home dan profile yang mengarahkan ke halaman Profile.
HomePage.vue
```html
<ion-content :fullscreen="true">
      <div>
      </div>
      <TabsMenu />
    </ion-content>
<script setup lang="ts">
import TabsMenu from '@/components/TabsMenu.vue';
</script>
```

TabsMenu.vue
```html
<ion-tabs>
        <ion-router-outlet></ion-router-outlet>

        <ion-tab-bar slot="bottom">
            <ion-tab-button tab="home" href="/home" layout="icon-top">
                <ion-icon :icon="home"></ion-icon>
                <ion-label>Home</ion-label>
            </ion-tab-button>

            <ion-tab-button tab="profile" href="/profile" layout="icon-top">
                <ion-icon :icon="person"></ion-icon>
                <ion-label>Profile</ion-label>
            </ion-tab-button>
        </ion-tab-bar>
    </ion-tabs>
```


## Halaman Profile

![Halaman Profil](gambar/profiledit.jpg)

Pada halaman profile menampilkan foto profil, nama akun, dan alamat email dari akun google yang terhubung.
```html
<div id="avatar-container">
    <ion-avatar>
        <img alt="Avatar" :src="userPhoto" @error="handleImageError" />
    </ion-avatar>
</div>

<!-- Data Profile -->
<ion-list>
    <ion-item>
        <ion-input label="Nama" :value="user?.displayName" :readonly="true"></ion-input>
    </ion-item>

    <ion-item>
        <ion-input label="Email" :value="user?.email" :readonly="true"></ion-input>
    </ion-item>
</ion-list>
```

Pada halaman tersebut juga mengimport dari halaman auth.ts untuk variabel isAuth karena perlu true untuk dapat mengakses halaman ini atau harus sudah login sehingga user.value sudah terisi supaya bisa didapatkan informasi akun. Contohnya yaitu dalam kode ini ada script untuk mengambil foto profil dari akun google.
```ts
import { useAuthStore } from '@/stores/auth';

const authStore = useAuthStore();
const user = computed(() => authStore.user);

const logout = () => {
    authStore.logout();
};

const userPhoto = ref(user.value?.photoURL || 'https://ionicframework.com/docs/img/demos/avatar.svg');

function handleImageError() {
    userPhoto.value = 'https://ionicframework.com/docs/img/demos/avatar.svg';
}
```