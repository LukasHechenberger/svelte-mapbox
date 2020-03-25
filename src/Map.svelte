<div bind:this={container}>
  {#if map}
  <slot></slot>
  {/if}
</div>

<style>
  div {
    width: 100%;
    height: 100%;
  }
</style>

<script>
  import { onMount, createEventDispatcher, setContext } from 'svelte'
  import { contextKey } from './mapbox.js'

  setContext(contextKey, {
    getMap: () => map,
    getMapbox: () => mapbox
	})

  const dispatch = createEventDispatcher()

  let container
  let map
  let mapbox

  export let options = {}
  export let accessToken
  export let style = 'mapbox://styles/mapbox/streets-v11'

  export function setCenter (center, zoom) {
    if (map) {
      map.setCenter(center)
      if (zoom && Number.isInteger(zoom)) {
        map.setZoom(zoom)
      }
    }
  }

  export function resize () {
    map && map.resize()
  }

  let lastBounds;
  function handleRebound(m = map) {
    const bounds = m.getBounds();
    const token = bounds.toString();

    if (token === lastBounds) return;

    lastBounds = token;
    dispatch('rebound', { map: m, bounds, token })
  }

  onMount(async () => {
    const mapboxModule = await import('mapbox-gl')
    mapbox = mapboxModule.default
    mapbox.accessToken = accessToken

    const link = document.createElement('link')
		link.rel = 'stylesheet'
		link.href = '//api.mapbox.com/mapbox-gl-js/v1.8.1/mapbox-gl.css'

    let el

		link.onload = () => {
      const optionsWithDefaults = Object.assign({
        container,
        style
      }, options)
      el = new mapbox.Map(optionsWithDefaults)

      el.on('dragend', () => dispatch('recentre', { center: el.getCenter() }))

      // So we can preload data while the map is loading
      handleRebound(el);

      el.on('zoomend', handleRebound.bind(null, el));
      el.on('moveend', handleRebound.bind(null, el));

      el.on('load', () => {
        map = el
        dispatch('ready')
      })
    }

    document.head.appendChild(link)

    return () => {
      map.remove()
      link.parentNode.removeChild(link)
    }
  })
</script>