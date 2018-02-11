<template>
  <div>
    <b-table
      @sort-changed="sort"
      :sort-by.sync="sortBy"
      :sort-desc.sync="isDescending"
      :no-local-sorting="true"
      :items="filteredGames"
      :fields="fields"
      responsive="md"
      striped
      v-if="games"
    >
      <template slot="imageUrl" slot-scope="data">
        <b-img width="75" :src="data.item.imageUrl" />
      </template>
      <template slot="average" slot-scope="data">
        <span class="badge" :class="['badge-' + getRatingColor(data.item.average, true)]">{{data.item.average | number}}</span>
      </template>
      <template slot="rating" slot-scope="data">
        <span class="badge" :class="['badge-' + getRatingColor(data.item.rating, true)]">{{data.item.rating | number}}</span>
      </template>
      <template slot="name" slot-scope="data">
        <a :href="'https://boardgamegeek.com/boardgame/' + data.item.id">{{data.item.name}}</a>
      </template>
      <template slot="weight" slot-scope="data">
        <span class="badge" :class="['badge-' + getWeightColor(data.item.weight)]">{{data.item.weight | number}}</span>
      </template>
      <template slot="mech" slot-scope="data">
        <ul>
          <li v-for="mechanism in data.item.mech">{{mechanism}}</li>
        </ul>
      </template>
      <tbody>
        <tr v-for="item in filteredGames" :key="item.id">
          <td v-if="hasHeader('rating') && singleUser">
            <span v-if="item.users[userId].rating" class="badge" :class="['badge-' + getRatingColor(item.users[userId].rating)]">{{item.users[userId].rating}}</span>
          </td>
          <td v-if="hasHeader('rating') && !singleUser">
            <span v-if="item.rating" class="badge" :class="['badge-' + getRatingColor(item.rating)]">
              {{item.rating | number}}
            </span>
            <i class="fa fa-users" aria-hidden="true"  v-if="item.rating" v-b-popover.hover="getUserRatings(item.users)" title="Individual Ratings"></i>
          </td>
          <td class="date" v-if="hasHeader('date')">
            <a>{{item.date}}</a>
          </td>
          <td v-if="hasHeader('weight')">
            <span class="badge" :class="['badge-' + getWeightColor(item.weight)]">{{item.weight | number}}</span>
          </td>
          <td v-if="hasHeader('playingtime')">{{item.playingtime}} mins</td>
          <td class="best-player" v-if="hasHeader('bggbestplayers')">{{item.bggbestplayers}}</td>
          <td class="num-plays" v-if="hasHeader('numplays') && singleUser">{{item.users[userId].numplays}}</td>
          <td class="num-plays" v-if="hasHeader('numplays') && !singleUser">
            {{item.numplays}}
            <i class="fa fa-users" aria-hidden="true" v-if="item.numplays" v-b-popover.hover="getUserPlays(item.users)" title="Individual Plays"></i>
          </td>
          <td class="wishlist-priority" v-if="hasHeader('wishlistpriority')">
            {{item.wishlistpriority | priority}}
          </td>
          <td class="comment" v-if="hasHeader('comment')">
            {{item.comment}}
          </td>
          <td class="mech" v-if="hasHeader('mech')">
            <ul>
              <li v-for="item in item.mech">{{item}}</li>
            </ul>
          </td>
        </tr>
      </tbody>
    </b-table>
    Item count: {{filteredGames.length}}
  </div>
</template>

<script>
import cookie from '~/components/cookie.js'
import filterItems from '~/components/filterItems.js'
var _ = require('lodash')

export default {
  computed: {
    filteredGames: function () {
      const games = filterItems(this.games, this.$store.state.filters)
      if (games.length) {
        const orderedGames = _.orderBy(games, [this.sortBy, 'average'], [this.isDescending ? 'desc' : 'asc', 'desc'])
        const firstrank = _.get(orderedGames, `0.rank`)
        const lastrank = _.get(orderedGames, `${orderedGames.length - 1}.rank`)
        if (orderedGames.length > 0 && (!firstrank && lastrank)) {
          while (!_.get(orderedGames[0], 'rank')) {
            orderedGames.push(orderedGames.shift())
          }
        }
        return orderedGames
      }
      return games
    }
  },
  created: function () {
    let users = cookie.get('username')
    users = users.split(',')
    if (users.length > 1 && users[users.length - 1]) {
      this.singleUser = false
    }
    this.userId = users[0]
  },
  data () {
    return {
      asc: true,
      isDescending: false,
      singleUser: true,
      userId: undefined,
      fields: [
        {
          key: 'imageUrl',
          tdClass: 'imageUrl',
          label: ' '
        }, this.hasHeader('rank') ? {
          sortable: true,
          key: 'rank',
          tdClass: 'rank'
        } : null, this.hasHeader('rating') ? {
          sortable: true,
          key: 'average',
          tdClass: 'average',
          label: 'Avg. Rating'
        } : null, this.hasHeader('rating') ? {
          sortable: true,
          key: 'rating',
          tdClass: 'rating'
        } : null, this.hasHeader('name') ? {
          sortable: true,
          key: 'name',
          tdClass: 'name'
        } : null, this.hasHeader('date') ? {
          sortable: true,
          key: 'date',
          tdClass: 'date'
        } : null, this.hasHeader('weight') ? {
          sortable: true,
          key: 'weight',
          tdClass: 'weight'
        } : null, this.hasHeader('playingtime') ? {
          sortable: true,
          key: 'playingtime',
          label: 'Length',
          tdClass: 'length',
          formatter: playingtime => `${playingtime} mins`
        } : null, this.hasHeader('bggbestplayers') ? {
          sortable: true,
          key: 'bggbestplayers',
          tdClass: 'bestplayers',
          label: 'Best # players',
          formatter: bestplayers => bestplayers || ''
        } : null, this.hasHeader('numplays') ? {
          sortable: true,
          key: 'numplays',
          tdClass: 'numplays',
          label: 'Plays'
        } : null, this.hasHeader('mech') ? {
          sortable: false,
          key: 'mech',
          tdClass: 'mech',
          label: 'Mechanisms'
        } : null, this.hasHeader('wishlistpriority') ? {
          sortable: true,
          key: 'wishlistpriority',
          tdClass: 'wishlistpriority',
          label: 'Priority',
          formatter: value => {
            switch (value) {
              case '1':
                return 'Must have'
              case '2':
                return 'Love to have'
              case '3':
                return 'Like to have'
              case '4':
                return 'Thinking about it'
              default:
                return "Don't buy this"
            }
          }
        } : null
      ].filter(i => !!i)
    }
  },
  filters: {
    number: function (value) {
      if (!value) return ''
      value = parseFloat(value)
      return value.toFixed(2)
    },
    priority: function (value) {
      switch (value) {
        case '1':
          return 'Must have'
        case '2':
          return 'Love to have'
        case '3':
          return 'Like to have'
        case '4':
          return 'Thinking about it'
        default:
          return "Don't buy this"
      }
    }
  },
  methods: {
    hasHeader: function (key, checkForHide) {
      if (!checkForHide) {
        return _.find(this.headers, ['key', key])
      } else {
        let header = _.find(this.headers, ['key', key])
        return header ? !header.hide : false
      }
    },
    getRatingColor: function (rating, roundDown) {
      return roundDown ? _.floor(rating) : _.ceil(rating)
    },
    getUserRatings: function (users) {
      let text = ''
      _.forEach(users, (user, userName) => {
        if (user.rating) {
          text += `${userName}: ${user.rating}\n`
        }
      })
      return text
    },
    getUserPlays: function (users) {
      let text = ''
      _.forEach(users, (user, userName) => {
        if (user.numplays) {
          text += `${userName}: ${user.numplays}\n`
        }
      })
      return text
    },
    getWeightColor: function (weight) {
      if (weight > 4) {
        return 'heavy'
      } else if (weight > 3.5) {
        return 'medium-heavy'
      } else if (weight > 3) {
        return 'medium'
      } else if (weight > 2.5) {
        return 'medium-light'
      } else {
        return 'light'
      }
    },
    sort: function (key) {
      if (!key) {
        return
      }
      if (key === this.sortBy) {
        this.asc = !this.asc
      } else {
        this.asc = true
        this.sortBy = key.sortBy
      }
    }
  },
  props: {
    defaultSort: {type: String},
    extFilters: { type: Object },
    games: { type: Array },
    headers: { type: Array },
    sortBy: {
      type: String,
      default: 'rank'
    }
  }
}
</script>

<style scoped>
.comment {
  text-align: left;
}

.table th {
  text-align: center;
}

.table td, .table th {
  padding: .25rem;
}

.table th:hover {
  cursor: pointer;
}

.table td {
  vertical-align: middle !important;
}

.name a {
  text-decoration: none;
}

.badge {
  font-size: 1rem;
}

.badge-10 {
  background: #00cc00;
}

.badge-9 {
  background: #33cc99;
}

.badge-8 {
  background: #66ff99;
}

.badge-7 {
  background: #99ffff;
}

.badge-6 {
  background: #9999ff;
}

.badge-5 {
  background: #cc99ff;
}

.badge-4 {
  background: #ff66cc;
}

.badge-3 {
  background: #ff6699;
}

.badge-2 {
  background: #ff3366;
}

.badge-1 {
  background: #ff0000;
}

.badge-0 {
  background: gray;
}

.badge-heavy {
  background: #800080;
  color: white;
}

.badge-medium-heavy {
  background: #a3529f;
  color: white;
}

.badge-medium {
  background: #c38bbf;
}

.badge-medium-light {
  background: #e2c5df;
}

.badge-light {
  background: #ffffff;
}
</style>
