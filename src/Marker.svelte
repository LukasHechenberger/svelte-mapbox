<script>
	import { getContext } from 'svelte'
  import { contextKey } from './mapbox.js'

	const { getMap, getMapbox } = getContext(contextKey)
  const map = getMap()
  const mapbox = getMapbox()

	export let lat
	export let lng
	export let label

	const popup = new mapbox.Popup({ offset: 25 })
		.setText(label)

	const marker = new mapbox.Marker()
		.setLngLat([lng, lat])
		.setPopup(popup)
		.addTo(map)
</script>
