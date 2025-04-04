<template>
  <div class="full">
    <div id="map">
      <div class="buttons">
        <template v-for="(category, index) in categories" :key="index">
          <button @click="onCategoryButtonClick(category)">
            {{ category.name }}
          </button>
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import { defineProps, onMounted, watch, ref, computed } from 'vue';
import { usePiniaStore } from '@/stores/pinia.js';
import apiClient from '@/api/axios.js';
import { useRoute,useRouter  } from 'vue-router';

const router = useRouter();
const route = useRoute();
// Pinia store에서 category 값을 가져옵니다
const piniaStore = usePiniaStore();

// 부모 컴포넌트로부터 전달된 주소를 받을 prop
const props = defineProps({
  address: {
    type: String,
    default: ''
  }
});

// map 객체를 ref로 정의
const map = ref(null);

// 신규 생성할 Category 에서 categoryId 를 들고 있을 데이터
let categoriesId = '';

// 지도에서 빈 지도에서 클릭시 저장할 배열
let markerMap = [];

// 네이버 지도 API 로드 및 지도 생성
onMounted(() => {
  const script = document.createElement("script");
  // NAVER CLOUD PLATFORM 네이버 클라우드 플랫폼 에서 네이버 지도 api key 발급 받아서 {본인의 id} 에 넣으면 구동 됩니다.
  script.src = "https://openapi.map.naver.com/openapi/v3/maps.js?ncpClientId={본인의 id}";
  script.async = true;
  script.defer = true;
  document.head.appendChild(script);

  script.onload = () => {
      // 네이버 지도 생성
      map.value = new window.naver.maps.Map("map", {
        center: new window.naver.maps.LatLng(37.5112, 127), // 한강 중심으로 하는 지도
        zoom: 13
      });

      // 클릭이벤트를 적용하여 경고창으로 위도 경도를 봅니다.
      naver.maps.Event.addListener(map.value, 'click', function(e){

        if (categoriesId)
        {
            const marker = new naver.maps.Marker({
            position: e.coord,
            map: map.value,
            icon: {
              url: "http://static.naver.com/maps2/icon/marker/marker_blue.png",  // 기본 마커 아이콘
              size: new naver.maps.Size(60, 80),  // 아이콘 크기 (너비: 40px, 높이: 60px)
              anchor: new naver.maps.Point(20, 60)  // 마커의 중심을 아이콘의 하단으로 맞추기
            }
          });

          // // 정보 창 내용 구성
          // const contentString = [
          //   '<div class="iw_inner" style="width: 360px; height: 120px; padding: 5px; font-size: 20px; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center;">',
          //     `   <h3>신규 장소를 생성하시겠습니까?</h3>`,
          //     `   <a href="/api/v1/new-place">새로운 장소 만들기</a>`,
          //   '</div>'
          // ].join('');

          // const infowindow = new naver.maps.InfoWindow({
          //   content: contentString,
          // });

          // 마커 클릭 시 정보 창을 띄우도록 이벤트 추가
          naver.maps.Event.addListener(marker, 'click', () => {

            // 기존 마커들과 정보창 삭제
            markerMap.forEach(markerData => {
              markerData.marker.setMap(null);  // 마커 삭제
              if (markerData.infowindow.getMap()) {
                markerData.infowindow.close();  // 열린 정보창 닫기
              }
            });
            markerMap = [];  // 배열 초기화

            // // 이미 정보 창이 열려있으면 닫고, 그렇지 않으면 열도록 처리
            // if (infowindow.getMap()) {
            //   infowindow.close();
            // } else {
            //   infowindow.open(map.value, marker);
            // }

            const point = new naver.maps.LatLng(e.coord.lat(), e.coord.lng());
              map.value.setCenter(point);  // 해당 위치로 지도 중심 이동
              map.value.setZoom(16);

            
            if(confirm('새로운 장소를 생성하시겠습니까?')){
              piniaStore.getLatitude(e.coord.lat());
              piniaStore.getLongitude(e.coord.lng());
              piniaStore.getCategoryId(categoriesId);

              router.push({ name: 'NewPlace' });

              
            }

            

            // 생성한 마커와 정보창을 markers 배열에 저장
            markerMap.push({
              marker: marker,
            });
          });
        }
    });
  }
});

// 최유진 필독!!!! 
// watch -> searchAddressToCoordinate 
// -> 서버에서 getMapping으로 좌표 가져옴
// -> 좌표 lng, lat에 저장 -> newPosition에 저장
// -> map.setCenter(newPosition)으로 위치 이동 -> 줌 확대
// -> 끝! 마커 표시필요함함
const searchAddressToCoordinate = async(address, map) => {
  try {
    const res = await apiClient.get('/geocode', {
      params: { address: address }
    })
    console.log('res.data:',res.data)
    const { lng, lat } = res.data

    if (!lat || !lng) {
      throw new Error("좌표가 유효하지 않습니다.")
    }

    const newPosition = new naver.maps.LatLng(lat, lng);

    // 지도 중심 이동
    map.setCenter(newPosition);

    // 줌 레벨 조정 (예: 17로 확대)
    map.setZoom(17);

  } catch (error) {
    alert('주소를 찾을 수 없습니다.')
    console.error(error)
  }  
}

// 마커와 관련된 정보창을 저장할 배열
let markers = [];

const categoryItems = computed(() => piniaStore.categories);

// 카테고리가 바뀔 때마다 실행되는 watch
watch(() => categoryItems.value, (newCategories) => {
  if (!map.value) return;

      // 기존 마커들과 정보창 삭제
      markers.forEach(markerData => {
        markerData.marker.setMap(null);  // 마커 삭제
        if (markerData.infowindow.getMap()) {
          markerData.infowindow.close();  // 열린 정보창 닫기
        }
      });
      markers = [];  // 배열 초기화

      markerMap.forEach(markerData => {
          markerData.setMap(null);  // 마커 삭제
      });
      markerMap = [];

  // 버튼을 다시 생성하여 위치를 조정
  createCategoryButtons(newCategories);
});

const createCategoryButtons = (categories) => {
  const buttonContainer = document.querySelector('.buttons');
  buttonContainer.innerHTML = ''; // 기존 버튼을 지웁니다

  // 새로운 버튼을 지도 위에 추가
  categories.forEach((category, index) => {
    const button = document.createElement('button');
    button.classList.add('btnCategory');
    button.innerText = category.name;
    button.addEventListener('click', () => onCategoryButtonClick(category));

    button.style.position = 'absolute';

    const buttonWidth = 150;  // 버튼의 고정된 너비 (동적 계산 대신 고정값 사용)
    const buttonHeight = 65;  // 버튼의 고정된 높이
    const horizontalSpacing = 160;  // 버튼 간 수평 간격
    const verticalSpacing = 80;    // 버튼 간 수직 간격
    
    // 버튼의 위치 계산 (5개씩 한 줄에 배치)
    const horizontalOffset = 10 + (index % 5) * horizontalSpacing;
    const verticalOffset = 10 + Math.floor(index / 5) * verticalSpacing;

    button.style.left = `${horizontalOffset}px`;
    button.style.top = `${verticalOffset}px`;
    button.style.height = `${buttonHeight}px`;
    button.style.width = `${buttonWidth}px`;

    // 글자 크기 설정
    button.style.fontSize = '26px';

    buttonContainer.appendChild(button);
  });
};

const onCategoryButtonClick = async (category) => {
  try {
    const response = await apiClient.get(`/categories/${category.categoryId}/places`);

    if (response.status === 200) {
      const places = response.data;

      categoriesId = category.categoryId;

      // 기존 마커들과 정보창 삭제
      markers.forEach(markerData => {
        markerData.marker.setMap(null);  // 마커 삭제
        if (markerData.infowindow.getMap()) {
          markerData.infowindow.close();  // 열린 정보창 닫기
        }
      });
      markers = [];  // 배열 초기화

      markerMap.forEach(markerData => {
          markerData.setMap(null);  // 마커 삭제
      });
      markerMap = [];

      // places가 비어있으면 알림 메시지 표시
      if (places.length === 0) {
        return; // 함수 종료
      }

      // 각 place에 대해 마커를 생성하고 배열에 저장
      places.forEach(place => {
        const marker = new naver.maps.Marker({
          position: new naver.maps.LatLng(place.latitude, place.longitude),
          map: map.value,
          title: place.name,
          icon: {
            url: "http://static.naver.com/maps2/icon/marker/marker_blue.png",  // 기본 마커 아이콘
            size: new naver.maps.Size(60, 80),  // 아이콘 크기 (너비: 40px, 높이: 60px)
            anchor: new naver.maps.Point(20, 60)  // 마커의 중심을 아이콘의 하단으로 맞추기
          }
        });


        // 정보 창 내용 구성
        const contentString = [
        '<div class="iw_inner" style="width: 350px; height: 300px; padding: 5px; font-size: 20px;">',
          `   <h3>${place.name}</h3>`,
          `   <p>${place.address}<br />`,
          `       <img src="${place.photoUrl || './img/example/hi-seoul.jpg'}" width="280" height="100" alt="${place.name}" class="thumb" /><br />`,
          `       ${place.phone || ''}<br />`,
          `       <a href="${`/api/v1/place/${place.placeId}` || '#'}">${place.name || '웹사이트 없음'}</a>`,
          '   </p>',
          '</div>'
        ].join('');

        const infowindow = new naver.maps.InfoWindow({
          content: contentString,
        });

        // 마커 클릭 시 정보 창을 띄우도록 이벤트 추가
        naver.maps.Event.addListener(marker, 'click', () => {
          // 이미 정보 창이 열려있으면 닫고, 그렇지 않으면 열도록 처리
          if (infowindow.getMap()) {
            infowindow.close();
          } else {
            infowindow.open(map.value, marker);
          }

          const point = new naver.maps.LatLng(place.latitude, place.longitude);
            map.value.setCenter(point);  // 해당 위치로 지도 중심 이동
            map.value.setZoom(16);
        });

        // 생성한 마커와 정보창을 markers 배열에 저장
        markers.push({
          marker: marker,
          infowindow: infowindow
        });
      });

    } else {
      console.error('API 요청 실패:', response.status);
    }
  } catch (error) {
    console.error('API 요청 실패:', error);
    alert('검색 실패. 다시 시도해주세요.');
  }
};
</script>

<style scoped>
.full * {
  box-sizing: border-box;
}

.full {
  position: absolute;
  top: 95px;
  left: 350px;
}

#map {
  width: 1550px;
  height: 990px;
}

.buttons {
  position: absolute;
  z-index: 1;
}
</style>
