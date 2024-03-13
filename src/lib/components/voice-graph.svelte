<script lang="ts">
  import { onMount } from "svelte"
  import * as d3 from "d3"
  interface DataItem {
    time: number
    pitch: number
  }
  let audio: AudioContext | null = null
  let width = 960
  let height = 400
  let data: DataItem[] = []

  // Must be called on analyser.getFloatTimeDomainData and audioContext.sampleRate
  // From https://github.com/cwilso/PitchDetect/pull/23
  function autoCorrelate(
    buffer: string | any[] | Float32Array,
    sampleRate: number
  ) {
    // Perform a quick root-mean-square to see if we have enough signal
    var SIZE = buffer.length
    var sumOfSquares = 0
    for (var i = 0; i < SIZE; i++) {
      var val = buffer[i]
      sumOfSquares += val * val
    }
    var rootMeanSquare = Math.sqrt(sumOfSquares / SIZE)
    if (rootMeanSquare < 0.01) {
      return -1
    }

    // Find a range in the buffer where the values are below a given threshold.
    var r1 = 0
    var r2 = SIZE - 1
    var threshold = 0.2

    // Walk up for r1
    for (var i = 0; i < SIZE / 2; i++) {
      if (Math.abs(buffer[i]) < threshold) {
        r1 = i
        break
      }
    }

    // Walk down for r2
    for (var i = 1; i < SIZE / 2; i++) {
      if (Math.abs(buffer[SIZE - i]) < threshold) {
        r2 = SIZE - i
        break
      }
    }

    // Trim the buffer to these ranges and update SIZE.
    buffer = buffer.slice(r1, r2)
    SIZE = buffer.length

    // Create a new array of the sums of offsets to do the autocorrelation
    var c = new Array(SIZE).fill(0)
    // For each potential offset, calculate the sum of each buffer value times its offset value
    for (let i = 0; i < SIZE; i++) {
      for (let j = 0; j < SIZE - i; j++) {
        c[i] = c[i] + buffer[j] * buffer[j + i]
      }
    }

    // Find the last index where that value is greater than the next one (the dip)
    var d = 0
    while (c[d] > c[d + 1]) {
      d++
    }

    // Iterate from that index through the end and find the maximum sum
    var maxValue = -1
    var maxIndex = -1
    for (var i = d; i < SIZE; i++) {
      if (c[i] > maxValue) {
        maxValue = c[i]
        maxIndex = i
      }
    }

    var T0 = maxIndex

    // Not as sure about this part, don't @ me
    // From the original author:
    // interpolation is parabolic interpolation. It helps with precision. We suppose that a parabola pass through the
    // three points that comprise the peak. 'a' and 'b' are the unknowns from the linear equation system and b/(2a) is
    // the "error" in the abscissa. Well x1,x2,x3 should be y1,y2,y3 because they are the ordinates.
    var x1 = c[T0 - 1]
    var x2 = c[T0]
    var x3 = c[T0 + 1]

    var a = (x1 + x3 - 2 * x2) / 2
    var b = (x3 - x1) / 2
    if (a) {
      T0 = T0 - b / (2 * a)
    }

    return sampleRate / T0
  }

  async function startAudio() {
    if (!navigator.mediaDevices.getUserMedia) {
      console.log("getUserMedia not supported on your browser!")
      return
    }
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
      audio = new window.AudioContext()
      const source = audio.createMediaStreamSource(stream)
      const analyser = audio.createAnalyser()
      source.connect(analyser)
      analyser.fftSize = 2048
      const bufferLength = analyser.fftSize
      const dataArray = new Float32Array(bufferLength)

      const maxPitchThreshold = 2000 // Define the maximum valid pitch threshold

      const getAudioData = () => {
        analyser.getFloatTimeDomainData(dataArray)
        if (audio) {
          let pitch = autoCorrelate(dataArray, audio.sampleRate)
          if (pitch > maxPitchThreshold) {
            pitch = -1 // Consider not showing this like the iOS app does
          }
          const now = Date.now()
          data = [...data, { time: now, pitch }]
          if (data.length > 10000) data.shift() // show only the last 10 seconds
        }
        window.requestAnimationFrame(getAudioData)
      }

      getAudioData()
    } catch (err) {
      console.error("The following getUserMedia error occurred:", err)
    }
  }

  onMount(() => {
    startAudio()
    const svg = d3
      .select("#graph")
      .attr("viewBox", `0 0 ${width} ${height}`)
      .style("width", "100%")
      .style("height", "auto")

    const xScale = d3
      .scaleTime()
      .domain([Date.now() - 1000, Date.now()])
      .range([0, width])

    const yScale = d3
      .scaleLinear()
      .domain([0, 1000]) // hardcode pitch range for visualization
      .range([height, 0])

    const line = d3
      .line<DataItem>()
      .x((d) => xScale(d.time))
      .y((d) => yScale(d.pitch))
      .curve(d3.curveLinear)

    const updateGraph = () => {
      xScale.domain([Date.now() - 10000, Date.now()])
      svg
        .selectAll("path")
        .data([data])
        .join("path")
        .attr("d", line)
        .attr("fill", "none")
        .attr("stroke", "steelblue")

      window.requestAnimationFrame(updateGraph)
    }

    updateGraph()
  })
</script>

<svg id="graph"></svg>
