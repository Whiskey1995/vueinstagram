<template>
  <div style="background:#222222">
    <!-- Instagram rainbow progress bar -->
    <vue-ins-progress-bar></vue-ins-progress-bar>

    <!-- Added snackbar -->
    <v-snackbar v-model="snackbar" :timeout="2000" top color="success">
      <img width="120" :src="modalUrl" alt />
      <p class="subtitle-2 mt-4">Awesome! You just added a new post.</p>
      <v-btn text color="white" @click="snackbar = false">Close</v-btn>
    </v-snackbar>

    <!-- Delete snackbar -->
    <v-snackbar v-model="delsnackbar" :timeout="2000" top color="red">
      <img width="120" :src="deleteUrl" alt />
      <p class="subtitle-2 mt-4">You Just delete a new project!</p>
      <v-btn text color="white" @click="delsnackbar = false">Close</v-btn>
    </v-snackbar>

    <div style="max-width:580px;" class="drageffect2 mx-auto">
      <!-- Stories skeleton loader -->
      <skeleton :loading="loading" :numbers="numbers"></skeleton>
      <!-- Stories skeleton loader -->

      <!-- Stories -->
      <story
        class="d-md-none d-block"
        :getTimeAgo="getTimeAgo"
        :projects="projects"
        :loading="loading"
      ></story>
      <!-- Stories -->
      <desktop-story
        class="d-md-block d-none"
        :getTimeAgo="getTimeAgo"
        :projects="projects"
        :loading="loading"
      ></desktop-story>
      <!-- Stories -->

      <v-divider class="mt-2 d-block d-sm-none"></v-divider>

      <!-- Feed -->
      <feed
        class="d-sm-none d-block"
        :getTimeAgo="getTimeAgo"
        :deleteProjects="deleteProjects"
        :toggleBottomSheet="toggleBottomSheet"
        :openBottomSheet="openBottomSheet"
        :projects="projects"
        :loading="loading"
      ></feed>
      <!-- Feed -->

      <!-- Feed -->
      <desktop-feed
        class="d-sm-block d-none"
        :getTimeAgo="getTimeAgo"
        :deleteProjects="deleteProjects"
        :toggleBottomSheet="toggleBottomSheet"
        :openBottomSheet="openBottomSheet"
        :projects="projects"
        :loading="loading"
      ></desktop-feed>
      <!-- Feed -->
    </div>
  </div>
</template>

<script>
// 如果我們的 Promise 成功完成的話，就可以呼叫 resolve() 來回傳 Promise，接著我們就可以用 then 來接續這個非同步執行完後要做的動作。

// 如果我們的 Promise 失敗的話，就可以呼叫 reject() 來回傳 失敗原因，接著我們就可以用 catch 來接住錯誤。

// var defer = new Promise(function(resolve, reject) {
//         if success, resolve()
//         if fail, reject()
// 	})
// 	.then(laterOn)
//   .catch(printError);

// var a = new Promise(function(resolve, reject) {
// 			setTimeout(function(){
// 				resolve('hello world');
//       }, 1000);

// 		});

var a = new Promise(function(resolve, reject) {
  setTimeout(function() {
    reject('OOps');
  }, 2000);
}).catch(function() {
  console.log(a);
});

import { db } from '../db';
import moment from 'moment';

import Skeleton from './Skeleton';
import Story from './Story';
import DesktopStory from './DesktopStory';
import Feed from './Feed';
import DesktopFeed from './DesktopFeed';

export default {
  props: {
    remove: Function,
  },
  components: {
    Skeleton,
    Story,
    Feed,
    DesktopStory,
    DesktopFeed,
  },
  data() {
    return {
      e1: 0,
      projects: [],
      loading: true,
      numbers: [5, 6, 7, 8, 9],
      delsnackbar: false,
      snackbar: false,
      overlay: false,
      power: null,
      insidedialog: false,
      circledialog: false,
      commentvalue: '',
      insvalue: 0,
      interval: {},
      insloadicon: './loading-icon.svg',
      modalUrl: './successpost.svg',
      deleteUrl: './delete.svg',
      vertical: true,
      emptyIcon: 'mdi-heart-outline',
      dialog: true,
      fullIcon: 'mdi-heart',
      halfIcon: 'mdi-heart-half-full',
      halfIncrements: false,
      hover: true,
      length: 5,
      rating: 0,
      readonly: false,
      size: 64,
      dense: false,
      bgColor: 'grey lighten-1',
      openBottomSheet: false,
    };
  },
  methods: {
    getTimeAgo(time) {
      if (!time) return '';
      return moment(time, 'YYYY-MM-DD').fromNow();
    },
    particlesEffect() {
      this.dialog = true;
    },
    sortBy(prop) {
      this.projects.sort((a, b) => (a[prop] < b[prop] ? -1 : 1));
      //     this.projects.sort((a,b)=>{
      //         if( a[prop]<b[prop]==true){
      //          return -1
      //         }
      //         else {
      //          return 1
      //         }
      //     })

      //1 就是rearrange array
      //? true 的時候進去下個動作
      //: false 的時候進去下個動作 也等於else
    },
    toggleBottomSheet() {
      this.openBottomSheet = !this.openBottomSheet;
    },
    deleteProjects(id) {
      // this.toggleBottomSheet();
      db.collection('projects')
        .doc(id)
        .delete()
        .then(() => {
          this.delsnackbar = true;
          this.getProjects();
        });
    },
    getProjects() {
      // req就是request(请求)
      // res就是response(响应)
      // 有请求就有响应，只是两个相对应的对象而已。
      // https://stackoverflow.com/questions/54773410/how-do-i-get-realtime-document-updates-from-firebase-firestore-with-change-type
      this.$insProgress.start();
      db.collection('projects').onSnapshot(res => {
        this.loading = true;
        this.dialog = true;
        // Returns an array of the documents changes since the last snapshot. If this is the first snapshot
        // , all documents will be in the list as added changes.
        const changes = res.docChanges();
        const projects = changes
          .map(change => {
            if (change.type !== 'added') return null;
            return {
              ...change.doc.data(),
              //return 成 物件
              id: change.doc.id,
            };
          })
          .filter(x => x);

        projects.sort((a, b) => {
          const aDate = moment(a.realtimeDate, 'YYYY-MM-DD HH:mm');
          const bDate = moment(b.realtimeDate, 'YYYY-MM-DD HH:mm');
          console.log(
            'bDate: ' +
              bDate +
              '\naDate: ' +
              aDate +
              '\ndiff: ' +
              bDate.diff(aDate),
          );
          return bDate.diff(aDate);
        });

        this.projects = [...projects, ...this.projects];
        this.loading = false;
        this.dialog = false;
        this.$insProgress.finish();
      });
    },
  },
  created() {
    this.getProjects();
  },
};
</script>

<style>
.loadingeffect {
  margin-top: 15px;
  margin-bottom: 20px;
}

.drageffect2 {
  margin-top: -20px;
  animation: drageffect2 0.5s;
}

.insloadicon {
  opacity: 0.4;
  transform: rotate(360deg);
  animation: rotatelogo 2s infinite;
}

.openeffect {
  transform: rotate(360deg);
  animation: rotatelogo 2s infinite;
}

@keyframes rotatelogo {
  0% {
    transform: rotate(360deg);
  }
  50% {
    transform: rotate(180deg);
  }
  100% {
    transform: rotate(0deg);
  }
}

@keyframes drageffect2 {
  0% {
    opacity: 0;
    margin-top: 50px;
  }
  100% {
    opacity: 1;
    margin-top: -20px;
  }
}

.avatarborder {
  border: 0px solid #d8d8d8 !important;
  transition: 0.4s;
  cursor: pointer;
}

.avatarborder:hover {
  border: 0px solid #d8d8d8 !important;
  transform: scale(1);
}

.avatarborder:active {
  border: 1px solid #d8d8d8 !important;
  transform: scale(0.9);
}
.complete {
  border-left: 4px solid #19f5df;
}

.ongoing {
  border-left: 4px solid #ff9100;
}

.overdue {
  border-left: 4px solid #d1413c;
}

.v-chip.complete {
  background: #3cd1c2;
}

.v-chip.ongoing {
  background: #ff7300;
}

.v-chip.overdue {
  background: #d1413c;
}

.theme--light.v-chip:not(.v-chip--active) {
  background: #2600ff;
}

#readmore {
  display: inline-block;
  font-size: 15px;
  color: #1c1c1d;
}

#readless {
  font-size: 15px;
  color: #1c1c1d;
}
</style>

<!-- 舊式寫法 -->
<!-- <v-layout row wrap justify-center>
        <v-flex  xs4 md3>
            <v-btn block tile outlined>CLick</v-btn>
        </v-flex>
          <v-flex xs4 md3>
            <v-btn block outlined>CLick</v-btn>
        </v-flex>
    </v-layout> -->

<!-- ｐ後面加a就代表是全部directions都加有加上 -->
<!-- tile 的意思就是沒有radius -->
