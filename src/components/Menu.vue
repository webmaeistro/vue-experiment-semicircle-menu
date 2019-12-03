<template>
  <aside :class="classes">
    <svg>
      <g v-if="debug" class="raster">
        <!-- the raster we put our svg on -->
        <line x1="1" y1="0" x2="1" y2="800" stroke="blue" />
        <line x1="201" y1="0" x2="201" y2="800" stroke="blue" />
        <line x1="401" y1="0" x2="401" y2="800" stroke="blue" />
        <line x1="601" y1="0" x2="601" y2="800" stroke="blue" />
        <line x1="801" y1="0" x2="801" y2="800" stroke="blue" />

        <line x1="0" y1="1" x2="800" y2="1" stroke="blue" />
        <line x1="0" y1="201" x2="800" y2="201" stroke="blue" />
        <line x1="0" y1="401" x2="800" y2="401" stroke="blue" />
        <line x1="0" y1="601" x2="800" y2="601" stroke="blue" />
        <line x1="0" y1="801" x2="800" y2="801" stroke="blue" />

        <!-- the circle we lay our menu items on -->
        <circle cx="400" cy="400" :r="menuItemRadius" stroke="blue" stroke-dasharray="4 6" fill="transparent" />

        <!-- The intersection lines that actually define where the menu items will be -->
        <template v-for="i in (2 * Math.floor((menuItemRadius * 2) / menuItemVerticalOffset))">
          <line
            :key="i"
            :x1="400 + intersectionWithCircle(-menuItemRadius + (i * (menuItemVerticalOffset / 2)), menuItemRadius, true)"
            :y1="400 - menuItemRadius + (i * (menuItemVerticalOffset / 2))"
            :x2="400 + intersectionWithCircle(-menuItemRadius + (i * (menuItemVerticalOffset / 2)), menuItemRadius, false)"
            :y2="400 - menuItemRadius + (i * (menuItemVerticalOffset / 2))"
            stroke="green"
            stroke-dasharray="2 6"
          />
        </template>

        <!-- All possible menu item locations, before culling -->
        <template v-for="{ x, y } in possibleMenuCoordinates">
          <line
            :key="`all-${x},${y}`"
            :x1="400 + x"
            :x2="400 + x"
            :y1="400 + y"
            :y2="400 + y"
            stroke="purple"
            stroke-width="7"
            stroke-linecap="round"
          />
        </template>

        <!-- Actual menu locations -->
        <template v-for="{ x, y } in menuCoordinates">
          <line
            :key="`menu-${x},${y}`"
            :x1="400 + x"
            :x2="400 + x"
            :y1="400 + y"
            :y2="400 + y"
            stroke="red"
            stroke-width="14"
            stroke-linecap="round"
          />
        </template>
      </g>

      <g class="circle-group">
        <template v-for="(arc, i) in arcs">
          <g :key="`arc${i}`" :class="`rotation-arc-group arc${i}`">
            <path :d="arc" />
          </g>
        </template>
      </g>

      <g class="menu-group" @click="toggleMenu">
        <line x1="-18" y1="-14" x2="18" y2="-14" />
        <g class="close-bar-1">
          <line x1="-18" y1="0" x2="18" y2="0" />
        </g>
        <g class="close-bar-2">
          <line x1="-18" y1="0" x2="18" y2="0" />
        </g>
        <line x1="-18" y1="14" x2="18" y2="14" />

        <!-- Making sure that we can click in a wide enough area, instead of only on the lines -->
        <rect x="-20" y="-20" width="40" height="40" fill="transparent" />
      </g>
    </svg>

    <div class="menu-links">
      <template v-for="menuItem of processedMenuItems">
        <div :key="menuItem.id" :style="menuItem.style">
          <a href="#">{{ menuItem.label }}</a>
        </div>
      </template>
    </div>
  </aside>
</template>

<script>
export default {
  name: 'MenuComponent',

  data() {
    return {
      debug: false,

      menuItemRadius: 200,
      menuItemVerticalOffset: 40,

      open: false,

      menuItems: [
        {
          label: 'Martin'
        },
        {
          label: 'mila'
        },
        {
          label: 'svarten'
        },
        {
          label: 'per'
        },
        {
          label: 'grå'
        },
        {
          label: 'Profiles'
        },
        {
          label: 'About'
        }
      ]
    }
  },

  computed: {
    arcs() {
      const multiplier = this.open ? 3.5 : 1
      const spacing = this.open ? 6 : 15
      const outerSpacing = this.open ? 112 : 110

      return [
        this.describeArc(0, 0, multiplier * (outerSpacing - 2 * spacing), 0, 60),
        this.describeArc(0, 0, multiplier * (outerSpacing - 2 * spacing), 0, 35),
        this.describeArc(0, 0, multiplier * (outerSpacing - 2 * spacing), 0, 70),
        this.describeArc(0, 0, multiplier * (outerSpacing - spacing), 0, 210),
        this.describeArc(0, 0, multiplier * outerSpacing, 0, 25),
        this.describeArc(0, 0, multiplier * outerSpacing, 0, 50)
      ]
    },

    classes() {
      return {
        'menu-component': true,
        open: this.open
      }
    },

    possibleMenuCoordinates() {
      const coordinates = []

      if ((this.menuItemRadius * 2) % this.menuItemVerticalOffset !== 0) {
        // If you get this error, you probably need to do more math to position the items correctly...
        // The code below assumes that the "middle" item is the slot at the very bottom, but when
        // you can't divide nicely we end up with some unusable space. You likely need to move everything down a bit
        throw new Error('menu diameter must be divisable by vertical offset')
      } 

      for (let y = -this.menuItemRadius + (this.menuItemVerticalOffset / 2); y <= this.menuItemRadius; y += this.menuItemVerticalOffset / 2) {
        coordinates.push({
          x: this.intersectionWithCircle(y, this.menuItemRadius, true),
          y
        })
      }

      for (let y = this.menuItemRadius - (this.menuItemVerticalOffset / 2); y >= -this.menuItemRadius; y -= (this.menuItemVerticalOffset / 2)) {
        coordinates.push({
          x: this.intersectionWithCircle(y, this.menuItemRadius, false),
          y
        })
      }

      return coordinates
    },

    menuCoordinates() {
      if (this.menuItems.length > (this.possibleMenuCoordinates.length / 2)) {
        throw new Error('You have too many menu items to properly render')
      }

      // If we need to fill up all the slots, use the top and bottom slots as well
      if (this.menuItems.length === (this.possibleMenuCoordinates.length / 2)) {
        return this.possibleMenuCoordinates.reduce(
          (acc, item, index) => {
            if (index % 2 === 1) {
              acc.push(item)
            }

            return acc
          },
          []
        )
      }

      const offset = Math.floor(this.possibleMenuCoordinates.length / this.menuItems.length)
      let bottomIndex = this.possibleMenuCoordinates.findIndex(({ y }) => y === this.menuItemRadius)
      let coordinates = []
      let currentOffset = 0
      
      if (this.menuItems.length % 2 === 1) {
        // If we have an odd number of items, use the bottom slot and branch out from there
        coordinates.push(this.possibleMenuCoordinates[bottomIndex])
      } else {
        // If we have an even number of items, use the two slots next to the bottom and go up
        coordinates.push(this.possibleMenuCoordinates[bottomIndex - 1])
        coordinates.push(this.possibleMenuCoordinates[bottomIndex + 1])

        currentOffset = 1
      }

      while (coordinates.length < this.menuItems.length) {
        currentOffset += offset

        coordinates.unshift(this.possibleMenuCoordinates[bottomIndex - currentOffset])
        coordinates.push(this.possibleMenuCoordinates[bottomIndex + currentOffset])
      }

      return coordinates
    },

    processedMenuItems() {
      return this.menuItems.map(
        (item, index) => {
          const { x, y } = this.menuCoordinates[index]

          const style = {
            top: `${y}px`,
            left: `${x}px`
          }

          return {
            ...item,
            id: index,
            style,
            route: '/'
          }
        }
      )
    }
  },

  methods: {
    // Thanks to upsidown (https://jsfiddle.net/upsidown/e6dx9oza/)
    polarToCartesian(centerX, centerY, radius, angleInDegrees) {
      const angleInRadians = (angleInDegrees - 90) * Math.PI / 180.0

      return {
        x: centerX + (radius * Math.cos(angleInRadians)),
        y: centerY + (radius * Math.sin(angleInRadians))
      }
    },

    describeArc(x, y, radius, startAngle, endAngle) {
      const start = this.polarToCartesian(x, y, radius, endAngle)
      const end = this.polarToCartesian(x, y, radius, startAngle)

      const arcSweep = endAngle - startAngle <= 180 ? '0' : '1'

      const d = [
        'M', start.x, start.y,
        'A', radius, radius, 0, arcSweep, 0, end.x, end.y
      ].join(' ')

      return d
    },

    intersectionWithCircle(y, radius, firstIntersection) {
      const x1 = -radius
      const x2 = radius

      const dx = x2 - x1
      const dy = 0
      const dr = Math.sqrt(Math.pow(dx, 2) + Math.pow(dy, 2))

      const matrixd = (x1 * y) - (x2 * y)

      const discriminant = (Math.pow(radius, 2) * Math.pow(dr, 2)) - Math.pow(matrixd, 2)

      let x = null
      if (discriminant < 0) {
        throw new Error('No such intersection exists')
      } else {
        const sign = (dy < 0 ? -1 : 1) * (firstIntersection ? -1 : 1)
        x = ((matrixd * dy) + (sign * dx * Math.sqrt(discriminant))) / Math.pow(dr, 2)
      }

      return x
    },

    toggleMenu() {
      this.open = !this.open
    }
  }
}
</script>

<style lang="scss" scoped>
.menu-component {
  position: absolute;
  top: 0;
  right: 0;
  z-index: 999;
  transform: translateX(0) translateY(0);
  transition: all .3s ease .2s;

  &.open {
    top: 50%;
    right: 50%;
    transform: translateX(50%) translateY(-50%);
    transition: all .3s ease;
  }
}

svg {
  height: 802px;
  width: 802px;

  line,
  path,
  rect {
    transition: opacity .1s linear, all .3s ease .1s;
  }

  g {
    transition: all .3s ease;
  }
}

.menu-group {
  cursor: pointer;

  line {
    stroke-width: 5;
    stroke: black;
    stroke-linecap: butt;
    opacity: 1;

    .open & {
      opacity: 0;
    }
  }
}

.rotation-arc-group {
  transform: rotate(0deg);
  transition: all .6s ease .1s;

  .open & {
    transform: rotate(var(--open-addition-rotation))
  }
}

.circle-group path {
  stroke-width: 3;
  stroke: black;
  stroke-linecap: round;
  fill: transparent;

  animation-name: lazy-rotation;
  animation-duration: var(--lazy-rotation-time);
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
  animation-direction: alternate;
}

.circle-group,
.menu-group {
  transition: all .3s ease .2s;
  transform: translate(701px, 51px);

  .open & {
    transition: all .3s ease;
    transform: translate(401px, 401px);
  }
}

.close-bar-1,
.close-bar-2 {
  line {
    .open & {
      opacity: 1;
    }
  }
}

.close-bar-1 {
  .open & {
    transform: rotate(45deg);
  }
}

.close-bar-2 {
  .open & {
    transform: rotate(-45deg);
  }
}

.arc0 {
  --initial-rotation: 175deg;
  --lazy-end-rotation: 190deg;
  --open-addition-rotation: 10deg;
  --lazy-rotation-time: 14s;
}

.arc1 {
  --initial-rotation: 270deg;
  --lazy-end-rotation: 260deg;
  --open-addition-rotation: 35deg;
  --lazy-rotation-time: 8s;
}

.arc2 {
  --initial-rotation: 50deg;
  --lazy-end-rotation: 70deg;
  --open-addition-rotation: -60deg;
  --lazy-rotation-time: 10s;
}

.arc3 {
  --initial-rotation: 75deg;
  --lazy-end-rotation: 85deg;
  --open-addition-rotation: 15deg;
  --lazy-rotation-time: 22s;
}

.arc4 {
  --initial-rotation: 240deg;
  --lazy-end-rotation: 265deg;
  --open-addition-rotation: -25deg;
  --lazy-rotation-time: 16s;
}

.arc5 {
  --initial-rotation: 115deg;
  --lazy-end-rotation: 125deg;
  --open-addition-rotation: -65deg;
  --lazy-rotation-time: 6s;
}

.menu-links {
  position: absolute;
  top: 50%;
  left: 50%;
  transition: all .2s ease;
  opacity: 0;

  .open & {
    opacity: 1;
    transition: all .3s ease .3s;
  }

  > * {
    position: absolute;
    white-space: nowrap;

    > * {
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translateX(-50%) translateY(-50%);

      color: black;
      font-size: xx-large;
    }
  }
}

@keyframes lazy-rotation {
  0% {
    transform: rotate(var(--initial-rotation));
  }

  100% {
    transform: rotate(var(--lazy-end-rotation));
  }
}
</style>
