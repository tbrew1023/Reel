<template>
  <div class="home-container">
    <svg id="logo" width="3329" height="3329" viewBox="0 0 3329 3329" fill="none" xmlns="http://www.w3.org/2000/svg">
      <rect id="outer" x="18" y="18" width="3293" height="3293" rx="1646.5" stroke="white" stroke-width="36"/>
      <path id="inner" d="M1936.95 833.751C1566.61 804.833 1273.25 950.227 1172.87 1026.54C1019.71 1126.95 702.371 1386.64 658.316 1622.12C614.26 1857.59 779.926 1908.43 868.266 1904.41C944.559 1894.66 1103 1829.02 1126.4 1644.49C1149.81 1459.97 1237.11 1356.46 1277.84 1327.77C1351.27 1275.56 1512.92 1197.64 1572.12 1303.67C1631.32 1409.7 1502.71 1575.07 1431 1644.49C1272.1 1778.18 974.617 2105.81 1055.84 2346.79C1157.38 2648.02 1425.31 2323.18 1577.28 2224.58C1704.63 2141.95 1998.9 2100.07 2071.18 2117.85C2126.25 2129.9 2221.24 2168.81 2153.78 2248.68C2069.46 2348.51 2060.85 2623.92 2449.78 2431.13C2838.7 2238.35 2652.84 1849.33 2449.78 1778.76C2246.71 1708.18 1754.53 1913.02 1714.95 1849.33C1675.37 1785.64 1670.21 1704.74 2035.04 1570.48C2399.87 1436.21 2399.87 869.898 1936.95 833.751Z" stroke="white" stroke-width="36"/>
    </svg>
    <!--router-link to="/editor">Dashboard</router-link-->
  </div>
</template>

<script>
export default {
    name: 'Home',
    components: {

    },
    data() {
        return {

        }
    },
    created() {

    },
    mounted() {
      //console.log('dashoffset: ', document.getElementById('logo-path').style.strokeDashoffset);
      console.log('rect length: ' + document.querySelector('#logo rect').getTotalLength());
      console.log('path length: ' + document.querySelector('#logo path').getTotalLength());
    },
    methods: {

    }
}
</script>

<style lang="scss" scoped>
.home-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  margin: 0px;
  padding: 0px;
  height: 100vh;
}

#logo {
  width: 300px;
  height: 300px;
  margin: 0px;
  padding: 0px;
  animation: fill-white 1s ease forwards 4.5s;
  margin-bottom: 36px;
  transition: 1s;
  //background: red;
}

#logo rect:nth-child(1) {
  stroke-dasharray: 10344.4667;
  stroke-dashoffset: 10344.4667;
  animation: line-animate 6s cubic-bezier(0.65, 0, 0.35, 1) forwards;
}

#logo path:nth-child(2) {
  stroke-dasharray: 9400.1387;
  stroke-dashoffset: 9400.1387;
  animation: line-animate 3s /*cubic-bezier(0.65, 0, 0.35, 1)*/ ease forwards;
  fill: #212225;
}

@keyframes line-animate {
  to {
      stroke-dashoffset: 0;
  }
}

@keyframes fill-white {
  from {
    fill: #212225;
  } to {
    fill: white;
    transform: translateY(-180px) scale(0.6);
  }
}

@keyframes fill-dark {
  to {
    fill: #212225;
  }
}
</style>