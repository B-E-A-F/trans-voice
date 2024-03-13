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
  let lastDotTime = 0 // Keep track of the last time a dot was added

  // Must be called on analyser.getFloatTimeDomainData and audioContext.sampleRate
  // From https://github.com/cwilso/PitchDetect/pull/23
  function autoCorrelate(
    buffer: string | any[] | Float32Array,
    sampleRate: number
  ) {
    // Perform a quick root-mean-square to see if we have enough signal
    let SIZE = buffer.length
    let sumOfSquares = 0
    for (let i = 0; i < SIZE; i++) {
      let val = buffer[i]
      sumOfSquares += val * val
    }
    let rootMeanSquare = Math.sqrt(sumOfSquares / SIZE)
    if (rootMeanSquare < 0.01) {
      return -1
    }

    // Find a range in the buffer where the values are below a given threshold.
    let r1 = 0
    let r2 = SIZE - 1
    let threshold = 0 // sensitivity threshold (lower is more sensitive)

    // Walk up for r1
    for (let i = 0; i < SIZE / 2; i++) {
      if (Math.abs(buffer[i]) < threshold) {
        r1 = i
        break
      }
    }

    // Walk down for r2
    for (let i = 1; i < SIZE / 2; i++) {
      if (Math.abs(buffer[SIZE - i]) < threshold) {
        r2 = SIZE - i
        break
      }
    }

    // Trim the buffer to these ranges and update SIZE.
    buffer = buffer.slice(r1, r2)
    SIZE = buffer.length

    // Create a new array of the sums of offsets to do the autocorrelation
    let c = new Array(SIZE).fill(0)
    // For each potential offset, calculate the sum of each buffer value times its offset value
    for (let i = 0; i < SIZE; i++) {
      for (let j = 0; j < SIZE - i; j++) {
        c[i] = c[i] + buffer[j] * buffer[j + i]
      }
    }

    // Find the last index where that value is greater than the next one (the dip)
    let d = 0
    while (c[d] > c[d + 1]) {
      d++
    }

    // Iterate from that index through the end and find the maximum sum
    let maxValue = -1
    let maxIndex = -1
    for (let i = d; i < SIZE; i++) {
      if (c[i] > maxValue) {
        maxValue = c[i]
        maxIndex = i
      }
    }

    let T0 = maxIndex

    // Not as sure about this part, don't @ me
    // From the original author:
    // interpolation is parabolic interpolation. It helps with precision. We suppose that a parabola pass through the
    // three points that comprise the peak. 'a' and 'b' are the unknowns from the linear equation system and b/(2a) is
    // the "error" in the abscissa. Well x1,x2,x3 should be y1,y2,y3 because they are the ordinates.
    let x1 = c[T0 - 1]
    let x2 = c[T0]
    let x3 = c[T0 + 1]

    let a = (x1 + x3 - 2 * x2) / 2
    let b = (x3 - x1) / 2
    if (a) {
      T0 = T0 - b / (2 * a)
    }

    return sampleRate / T0
  }

  let lastDataUpdateTime = 0 // keep track of the last update time

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

      const maxPitchThreshold = 600 // Ignore Hz values above this threshold
      const minPitchThreshold = 20 // Ignore Hz values below this threshold

      const getAudioData = () => {
        analyser.getFloatTimeDomainData(dataArray)
        if (audio) {
          let pitch = autoCorrelate(dataArray, audio.sampleRate)
          if (pitch > maxPitchThreshold || pitch < minPitchThreshold) {
            pitch = -1
          }
          const now = Date.now()
          // Check if at least 300ms have passed since the last update
          if (now - lastDataUpdateTime > 300) {
            data = [...data, { time: now, pitch }]
            if (data.length > 1000) data.shift() // Show only the last 10 seconds
            lastDataUpdateTime = now
          }
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

    const yScale = d3.scaleLinear().domain([50, 500]).range([height, 0])

    const drawDots = (data: DataItem[]) => {
      svg
        .selectAll("circle")
        .data(data)
        .join("circle")
        .attr("cx", (d) => xScale(d.time))
        .attr("cy", (d) => yScale(d.pitch))
        .attr("r", 3) // Size of the dots
        .attr("fill", "steelblue")
    }

    // connect dots if they are close enough in time (3 seconds)
    const connectDots = (data: any[]) => {
      // Filter out isolated dots at the start and end
      const filteredData = data.filter(
        // Keep if not first or last point, or if adjacent points are within the time range
        (d, i, arr) =>
          (i > 0 && arr[i].time - arr[i - 1].time <= 3000) ||
          (i < arr.length - 1 && arr[i + 1].time - arr[i].time <= 3000)
      )

      const linesData = filteredData.filter(
        (_, i, arr) => i > 0 && arr[i].time - arr[i - 1].time <= 3000
      )

      const line = d3
        .line<DataItem>()
        .defined((d) => d.pitch > -1) // don't show lines for points with no valid pitch
        .x((d) => xScale(d.time))
        .y((d) => yScale(d.pitch))
        .curve(d3.curveLinear)

      svg
        .selectAll("path")
        .data([linesData])
        .join("path")
        .attr("d", line)
        .attr("fill", "none")
        .attr("stroke", "steelblue")
    }

    const updateGraph = () => {
      xScale.domain([Date.now() - 10000, Date.now()])
      const newData = data.filter((d) => d.time >= Date.now() - 10000)

      drawDots(newData)
      connectDots(newData)

      window.requestAnimationFrame(updateGraph)
    }

    updateGraph()
  })
</script>

<svg id="graph"></svg>
