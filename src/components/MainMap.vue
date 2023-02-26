<!-- eslint-disable no-unused-vars -->
<!-- eslint-disable no-undef -->
<template>
  <div id="map" />
</template>

<script setup>
import { DynamoDBClient, ScanCommand } from '@aws-sdk/client-dynamodb'
import { Loader } from '@googlemaps/js-api-loader'
import { onMounted, ref, watch } from 'vue'
// Create an Amazon DynamoDB service client object.

const emit = defineEmits(['zoom', 'info'])
const zoomLevel = ref(0)
watch(zoomLevel, () => {
  const markers = document.getElementsByClassName('busy-hex')
  const text = document.getElementsByClassName('busy-count')
  const height = Math.pow(1.87091, zoomLevel.value - 14.9487) + 2
  for (let i = 0; i < markers.length; i++) {
    markers[i].style.height = `${height}vh`
    markers[i].style.transform = `translateY(${height / 2}vh)`
    text[i].style.transform = `scale(${height / 4})`
    console.log(markers[i].style.transform)
    if (zoomLevel.value <= 13.5) {
      markers[i].style.opacity = (200 * (zoomLevel.value - 13)) / 100
    } else {
      markers[i].style.opacity = 1
    }
  }
})

onMounted(() => {
  const loader = new Loader({
    apiKey: import.meta.env.VITE_MAP_API,
    version: 'beta',
    libraries: ['marker', 'drawing', 'places']
  })

  const ddbClient = new DynamoDBClient({
    region: 'us-east-2',
    credentials: {
      accessKeyId: import.meta.env.VITE_AWS_KID,
      secretAccessKey: import.meta.env.VITE_AWS_SAK
    }
  })

  loader.load().then(async () => {
    const map = new google.maps.Map(document.getElementById('map'), {
      center: { lat: 39.1145, lng: -84.511734 },
      zoom: 15,
      gestureHandling: 'greedy',
      mapId: import.meta.env.VITE_MAP_STYLE,
      disableDefaultUI: true
    })

    const data = await ddbClient.send(
      new ScanCommand({
        TableName: 'establishments'
      })
    )

    for (let i = 0; i < data.Count; i += 1) {
      const drawContainer = document.createElement('div')
      drawContainer.className = 'busy-hex'
      // drawContainer.textContent = `${Number(data.Items[i]['curr_cap']['N'])}`
      let capPercent =
        (Number(data.Items[i]['curr_cap']['N']) / Number(data.Items[i]['max_cap']['N'])) * 100
      drawContainer.style.backgroundColor = `rgb(${2 * (255 / 100) * capPercent}, ${
        capPercent < 50 ? 200 : (capPercent - 100) * (-2 * (200 / 100))
      }, 0)`
      drawContainer.style.clipPath = 'polygon(25% 5%, 75% 5%, 100% 50%, 75% 95%, 25% 95%, 0% 50%)'
      const text = document.createElement('p')
      text.textContent = data.Items[i]['curr_cap']['N']
      drawContainer.appendChild(text).classList.add('busy-count')
      const markerView = new google.maps.marker.AdvancedMarkerView({
        map,
        position: {
          lat: Number(data.Items[i]['lat']['N']),
          lng: Number(data.Items[i]['long']['N'])
        },
        content: drawContainer,
        title: `Current capacity: ${Number(data.Items[i]['curr_cap']['N'])}/${Number(
          data.Items[i]['max_cap']['N']
        )}`,
        collisionBehavior: 'OPTIONAL_AND_HIDES_LOWER_PRIORITY'
      })

      const request = {
        query: data.Items[i]['name']['S'],
        locationBias: {
          center: {
            lat: Number(data.Items[i]['lat']['N']),
            lng: Number(data.Items[i]['long']['N'])
          },
          radius: 30
        },
        fields: ['place_id']
      }
      const service = new google.maps.places.PlacesService(map)
      service.findPlaceFromQuery(request, (results, status) => {
        if (status === google.maps.places.PlacesServiceStatus.OK) {
          for (var i = 0; i < results.length; i++) {
            const place_id = results[i]['place_id']
            const service2 = new google.maps.places.PlacesService(map)
            service2.getDetails({ placeId: place_id, fields: ['ALL'] }, (res, stat) => {
              if (stat === 'OK') {
                drawContainer.addEventListener('mouseup', () => {
                  emit('info', res)
                })
              }
            })
          }
        }
      })
    }

    map.addListener('zoom_changed', () => {
      zoomLevel.value = map.getZoom()
      emit('zoom', map.getZoom())
    })
    map.addListener('click', () => {
      emit('info', {})
    })
  })
})
</script>

<style>
#map {
  height: 100vh;
  width: 100vw;
  z-index: 0;
}

.busy-hex {
  height: 3.033vh;
  transform: translateY(1.5165vh);
  display: flex;
  align-items: center;
  justify-content: center;
  aspect-ratio: 1/1;
  color: white;
  border-radius: 10px;
  font-family: 'Consolas';
}

.busy-count {
  transform: scale(0.75825);
}

.busy-hex:hover {
  opacity: 75%;
  cursor: pointer;
}
</style>
