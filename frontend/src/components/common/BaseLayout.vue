<template>
  <div class="left">   
    <div class="mappride">Mappride</div>

    <div class="leftRectangle"></div>

    <RouterLink :to="{ name: 'Main' }" class="map">
    <img class="imgMap" src="/src/assets/images/public/image-120.png" />
    Map
    </RouterLink>

    <RouterLink :to="{ name: 'Mypage' }" class="my-page">
    <img class="imgPage" src="/src/assets/images/public/image-130.png" />
    My Page
    </RouterLink>

    <RouterLink :to="{ name: 'Categories' }" class="my-categories">
    <img class="imgCategory" src="/src/assets/images/public/image-140.png"/>
    Categories
    </RouterLink>
  </div>

  <div class="top">
    <div class="topLine"></div>
    <img @click="handleClick" class="btnNotice" src="/src/assets/images/public/image-150.png" />

    <button v-if="route.name == 'Main'" class="myCategories" @click="showMyCategories()">My Categories</button>

    <div v-if="route.name === 'Main'">
      <!-- <select v-model="selectedOption" class="btnDD">
          <option value="address">Address</option>
          <option value="user">User</option>
      </select> -->
      <div class="lblName">user</div>
      <input :value="txtSearchModel" class="txtSearch" type="text" placeholder="Search..." @input="txtChanging">
      <ul v-if="filteredData.length > 0" class="lookupResults">
        <li
          v-for="(item, index) in filteredData" :key="index" @click="onItemSelect(item)" class="lookupItem">
          {{ "이름: " + item.name + ' / ' + "닉네임: " + item.nickname }}
        </li>
      </ul>
      <button class="btnSearch" >search</button>
    </div>

    <div v-if="route.name === 'Categories' || route.name === 'Place'">
      <div class="lblName">name</div>
      <input :value="txtSearchModel" class="txtSearch" type="text" placeholder="Search..." @input="txtChanging"/>
      <button class="btnSearch" @click="btnSearchClick">search</button>
    </div>

    <button class="logout" v-if="route.path !== '/api/v1'" @click="logout"> logout </button>
    
    <div v-if="route.name === 'Login'">
      <input class="txtSearch" type="text" placeholder="Search..." />
      <button class="btnSearch"@click="onsearch">search</button>
      <button class="btnLogin">login</button>``
    </div>
  </div>

  <!-- <RouterView :searchQuery="searchQuery" /> -->
  <RouterView></RouterView>
</template>
    
<script setup>
  import { ref, reactive, onMounted } from 'vue';
  import { useRoute, useRouter } from 'vue-router';
  import { usePiniaStore } from '@/stores/pinia.js';
  import apiClient from '@/api/axios.js';
  // import { useSearchStore } from '@/stores/pinia.js'; // Pinia store import

  // Pinia store에 연결된 searchQuery
  // const store = useSearchStore();
  // const searchQuery = ref(store.searchQuery);

  const router = useRouter();
  const route = useRoute();
  const selectedOption = ref('address');
  const txtSearchModel = ref('');
  const txtMemberId = ref('');
  const filteredData = ref([]);
  const address = reactive({ addressValue: '' });
  const categories = reactive({ categoriesValue: '' });
  const piniaStore = usePiniaStore();

  // 검색어 상태 설정
  const emitSearch = () => {
    store.setSearchQuery(searchQuery.value); // 검색어 상태 업데이트
    console.log("검색 실행:", searchQuery.value);
  };

  const logout = () => {
    const isConfirmed = window.confirm("로그아웃 하시겠습니까?");
    if (!isConfirmed) return; // 취소 시 아무 동작 안 함

    localStorage.removeItem('accessToken'); // 토큰 삭제
    router.push({ name: 'Login' }); // 로그인 페이지로 이동
  };

  const txtChanging = async (event) => {
    try {
      txtSearchModel.value = event.target.value;

      console.log(txtSearchModel.value);

      if (route.name === 'Main') {
        const response = await apiClient.get(`/members/name?name=${txtSearchModel.value}`);

        if (response.status === 200) {
          filteredData.value = response.data;
        } else {
          console.error('API 요청 실패:', response.status);
        }
      }
    } catch (error) {
      console.error('API 요청 실패:', error);
      alert('검색 실패. 다시 시도해주세요.');
    }
  };

  const showMyCategories = async () => {
    piniaStore.getMyCategories()
  };

  const onItemSelect = async (item) => {
    txtSearchModel.value = item.name;
    txtMemberId.value = item.id;
    filteredData.value = [];

    try {
      if (route.name === 'Main') {
        if (!txtMemberId.value) return;
        
        piniaStore.getCategory(txtMemberId.value);
      }
    } catch (error) {
      console.error('API 요청 실패:', error);
      alert('검색 실패. 다시 시도해주세요.');
    }
  };

  const btnSearchClick = async () => {
    try {
      if(route.name === 'Categories'){

        if (!txtSearchModel.value) return;

        piniaStore.getMyCategoryName(txtSearchModel.value);
      }
      else if (route.name === 'Main') {

        if (!txtMemberId.value) return;

        piniaStore.getCategory(txtMemberId.value);

      } 

      // 카테고리 검색 추가
      // else if (selectedOption.value === 'category') {  
      //   if (!txtSearchModel.value) return;

      //   const response = await apiClient.get(`/categories/search?query=${txtSearchModel.value}`);

      //   if (response.status === 200) {
      //     categories.categoriesValue = response.data;
      //   } else {
      //     console.error('카테고리 검색 API 요청 실패:', response.status);
      //   }
      // }
    } catch (error) {
      console.error('API 요청 실패:', error);
      alert('검색 실패. 다시 시도해주세요.');
    };
  }
</script>
    
<style scoped>
/* Left */
  .left {
    position: relative;  /* 부모 요소는 relative로 설정 */
    background: #ffffff;
    left: 0;
    top: 0;
    width: 350px;
    height: 1080px;
  }
  .mappride {
    color: #000000;
    text-align: left;
    font-family: "Stylish-Regular", sans-serif;
    font-size: 30px;
    font-weight: 400;
    position: absolute;
    left: 78.27px;
    top: 31.64px;
    width: 186.84px;
    height: 40.08px;
    -webkit-text-stroke: 1px #ffffff;
  }    
  .leftRectangle {
    background: rgba(255, 255, 255, 0);
    border-style: solid;
    border-color: #d9d9d9;
    border-width: 1px;
    width: 336.87px;
    height: 1080px;
    position: absolute;
  }
  .map, .my-page, .my-categories{
    color: #000000;
    text-align: center;
    font-family: "Stylish-Regular", sans-serif;
    font-size: 25px;
    font-weight: 400;
    position: absolute;
    width: 318.14px;
    height: 104.41px;
    display: flex;
    align-items: center;
    justify-content: center;
    -webkit-text-stroke: 1px #ffffff;
    background-color: transparent;
    border: none;
    cursor: pointer;
  }
  .map {
    top: 335.39px;
    left: 20.14px;
  }
  .my-page {
    top: 439.80px;
    left: 20.14px;
  }
  .my-categories {
    top:  544.21px;
    left: 20.14px;
  }
  .map:hover {
    background-color: #f0f0f0;
  }
  .my-page:hover {
    background-color: #f0f0f0;
  }
  .my-categories:hover {
    background-color: #f0f0f0;
  }
  .logout {
    color: #000000;
    text-align: center;
    font-family: "Stylish-Regular", sans-serif;
    font-size: 20px;
    font-weight: 400;
    position: absolute;
    width: 80px;
    height: 50px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: transparent;
    border: none;
    cursor: pointer;
    left: 1404px;
    top: 30px;
  }
  .logout:hover {
    background-color: #f0f0f0;
  }

  button:hover {
    background-color: #f0f0f0;
  }
  .imgMap {
    width: 44.02px;
    height: 44.02px;
    position: absolute;
    left: 25.64px;
    object-fit: cover;
    aspect-ratio: 1;
  }
  .imgPage {
    width: 29.75px;
    height: 29.75px;
    position: absolute;
    left: 32.78px;
    object-fit: cover;
    aspect-ratio: 1;
  }
  .imgCategory {
    width: 30.94px;
    height: 30.94px;
    position: absolute;
    left: 32.86px;
    object-fit: cover;
    aspect-ratio: 1;
  }    

/* top  */
  .top {
    position: relative;
    background: #ffffff;
    position: absolute;
    left: 350px;
    top: 0;
    right: 0;
    bottom: 0;
  }
  .topLine {
    background: #ffffff;
    border-style: solid;
    border-color: #d3d3d3;
    border-width: 0px 0px 1px 0px;
    width: 1550px;
    height: 90px;
    position: absolute;
  }  
  .btnDD {
    background: #ffffff;
    border-style: solid;
    border-color: #d2d2d2;
    border-width: 1px;
    width: 165px;
    height: 45px;
    position: absolute;
    left: 168px;
    top: 27px;
    padding: 5px 10px; /* 텍스트와 경계선 사이에 여백을 추가 */
    font-size: 16px; /* 텍스트 크기 설정 */
    color: #000000; /* 텍스트 색상 설정 */
    cursor: pointer; /* 드롭다운 클릭 시 포인터 커서 */
  }  
  .btnDD option {
    padding: 10px; /* 옵션 항목에 여백 추가 */
  }
  .lblName {
    background: #ffffff; /* 박스 배경 색 */
    border-style: solid;
    border-color: #d2d2d2;
    border-width: 1px;
    width: 165px;
    height: 42px;
    display: flex;
    justify-content: center; /* 텍스트 수평 중앙 정렬 */
    align-items: center; /* 텍스트 수직 중앙 정렬 */
    font-family: "Stylish-Regular", sans-serif;
    font-size: 20px;
    font-weight: 400;
    color: #000000; /* 텍스트 색상 */
    -webkit-text-stroke: 1px #d2d2d2; /* 텍스트 테두리 */
    position: absolute;
    left: 168px;
    top: 27px;
  }
  .txtSearch {
    background: #ffffff;
    border-style: solid;
    border-color: #d2d2d2;
    border-width: 1px;
    width: 460px;
    height: 22px;
    position: absolute;
    left: 352px;
    top: 27px;
    padding: 10px 15px; /* 텍스트와 경계선 사이에 여백을 추가 */
    font-size: 16px; /* 텍스트 크기 설정 */
    color: #000000; /* 텍스트 색상 설정 */
  }  
  .txtSearch::placeholder {
    color: #d2d2d2; /* placeholder 텍스트 색상 설정 */
  }
  .lookupResults {
    left: 352px;
    top: 56px;
    overflow-y: auto;
    border: 1px solid #ccc;
    border-radius: 4px;
    position: absolute;
    background-color: white;
    width: 100%;
    z-index: 2;
    width: 450px;
  }

  .lookupItem {
    padding: 8px;
    cursor: pointer;
  }

  .lookupItem:hover {
    background-color: #f0f0f0;
  }

  .loading {
    font-size: 14px;
    color: gray;
    padding-top: 5px;
  }
  .btnSearch {
    color: #000000;
    text-align: center; /* 텍스트를 중앙 정렬 */
    font-family: "Stylish-Regular", sans-serif;
    font-size: 20px;
    font-weight: 400;
    position: absolute;
    left: 867px;
    top: 27px;
    border-width: 1px;
    width: 165px;
    height: 45px;
    background-color: transparent; /* 배경을 투명하게 설정 */
    border: 2px solid #d2d2d2; /* 테두리 추가 (두께 2px, 색상 #d2d2d2) */
    cursor: pointer; /* 마우스를 올렸을 때 커서가 포인터로 바뀌게 설정 */
    -webkit-text-stroke: 1px #d2d2d2;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .btnNotice {
    width: 46px;
    height: 46px;
    position: absolute;
    left: 1224px;
    top: 22px;
    object-fit: cover;
    aspect-ratio: 1;
  }
  .btnLogout, .btnLogin {
    color: #000000;
    text-align: left;
    font-family: "Stylish-Regular", sans-serif;
    font-size: 20px;
    font-weight: 400;
    position: absolute;
    left: 1404px;
    top: 27px;
    width: 120px;
    height: 45px;
    background-color: transparent; /* 배경을 투명하게 설정 */
    border: 2px solid #d2d2d2; /* 테두리 추가 (두께 2px, 색상 #d2d2d2) */
    cursor: pointer; /* 마우스를 올렸을 때 커서가 포인터로 바뀌게 설정 */
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .myCategories {
    width: 120px;
    height: 60px;
    font-size: 20px;
    font-weight: 400;
    position: absolute;
    left: 20px;
    top: 25px;
  }
  .myCategories:hover {
    background-color: #d2d2d2;
  }

</style>