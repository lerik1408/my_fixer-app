<template>
  <div>
    <aside-component v-bind:active="active"></aside-component>
    <header-component v-bind:breadcrumbs="breadcrumbs"></header-component>
    <main class="main">
      <section class="main__wrap">
        <form-component @category="search" @loc="location"></form-component>

          <div class="content">
            <p @click="switchTab" class="content__tab">{{textTab}}</p>
            <div v-if="!active.map" class="people" v-scroll="onScroll">
              <people-component
                v-for="people in peoples"
                v-bind:people="people"
                v-bind:key="people._id"
              ></people-component>
            </div>
            <map-component v-if="active.map" v-bind:people="people"></map-component>
        </div>
      </section>
    </main>
  </div>
</template>

<script>
import asideComponent from '../../components/aside';
import headerComponent from '../../components/header';
import formComponent from '../components/form';
import peopleComponent from '../components/people';
import mapComponent from '../components/map';
import api from '../../shared/services/api.axios';

export default {
  name: 'search',
  data() {
    return {
      peoples: undefined,
      active: {
        search: true,
        map: false,
      },
      query: {
        name: '',
        category: '0',
        sort: '0',
        skip: 0,
      },
      breadcrumbs: [
        { id: 1, text: 'Home', link: '/index' },
        { id: 2, text: 'Search', link: '/search' },
      ],
    };
  },
  components: {
    asideComponent,
    headerComponent,
    formComponent,
    peopleComponent,
    mapComponent,
  },
  computed: {
    textTab() {
      if (!this.active.map) {
        return 'Show map';
      }
      return 'Show result';
    },
  },
  mounted() {
    console.log(this.$router.history.current.query)
    const queryShared = this.$router.history.current.query
    // Check void obj
    if (Object.keys(queryShared).length !== 0) {
      this.query.name = queryShared.name;
      this.query.category = queryShared.category;
      this.query.sort = queryShared.sort;
    }
    api.get(`/api/search/people?name=${this.query.name}&category=${this.query.category}&sort=${this.query.sort}&skip=${this.query.skip}`).then((res) => {
      this.peoples = res.data.allPeople;
    });
  },
  methods: {
    onScroll(e, param){
      console.log(param);
      console.log(e);
      // console.log(e.target.scrollHeight-param.scrollTop===418)
      if(e.target.scrollHeight-param.scrollTop===418){
        console.log('end')
        this.query.skip+=1
        api.get(`/api/search/people?name=${this.query.name}&category=${this.query.category}&sort=${this.query.sort}&skip=${this.query.skip}`).then((res) => {
          console.log(res)
        this.peoples = res.data.allPeople;
      });
      }
    },
    search(params) {
      this.query.skip = 0;
      this.query = params;
      this.$router.push({ path: '', query: this.query });
      api.get(`/api/search/people?name=${this.query.name}&category=${this.query.category}&sort=${this.query.sort}&skip=${this.query.skip}`).then((res) => {
        this.peoples = res.data.allPeople;
      });
    },
    switchTab() {
      this.active.map = !this.active.map;
    },
    location(e) {
      if(e.target.checked){
         api.get('/api/search/location').then((res)=>{
           console.log(res.data)
           this.peoples = res.data
         })
      }else{
      api.get(`/api/search/people?name=${this.query.name}&category=${this.query.category}&sort=${this.query.sort}&skip=${this.query.skip}`).then((res) => {
      this.peoples = res.data.allPeople;
    });
      }
    }
  },
};
</script>

<style lang="sass" scoped>
@import '../../shared/style/variables.sass'
@import '../../shared/style/respond.sass'
main
    display: flex
    margin-left: 80px
    background-color: $grayMain
    @include respond_tablet
        margin-left: 0
.main__wrap
    display: flex
    width: 90%
    height: 556px
    margin: auto
    margin-top: 25px
    border-radius: 4px
    background-color: #fff
    box-shadow: 0 0 30px rgba(153, 163, 174, 0.06)
    @include respond_bigTablet
        width: 100%
    @include respond_tablet
        width: 97%
        height: 100%
        flex-direction: column
.people
  &::-webkit-scrollbar
    width: 8px
  &::-webkit-scrollbar-track
    border-radius: 4px
    background: rgba(56, 68, 99, 0.05)
  &::-webkit-scrollbar-thumb
    border-radius: 4px
    background: rgba(19, 66, 189, 0.1)
.content
  margin-top: 40px
  margin-left: 41px
  width: 100%
  @include respond_tablet
    margin: 0
.content__tab
  text-transform: capitalize
  font-family: $Roboto
  color: $navyBlue
  font-weight: 600
  cursor: pointer
.people
    width: 95%
    height: 418px
    margin-top: 31px
    display: flex
    flex-wrap: wrap
    overflow: auto
    @include respond_tablet
      width: 100%
.pagination
  display: flex
  justify-content: center
  align-items: center
  margin-top: 15px
.pagination>a
  margin-right: 10px
  font-size: 20px
  color: $navyBlue
  text-decoration: none
</style>
