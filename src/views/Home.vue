<template>
  <div class="layout">
    <Layout>
      <Header :style="{ position: 'fixed', width: '100%' }" class="nav">
        <Menu mode="horizontal" theme="dark" active-name="1">
          <div class="layout-nav">
            <router-link to="/">
              <MenuItem name="Home"><Icon type="ios-navigate" />Home</MenuItem>
            </router-link>
            <MenuItem name="Fav"><Icon type="ios-bookmark" />Favorite</MenuItem>
            <MenuItem name="0">
              <Input
                search
                placeholder="Enter something..."
                v-model="keyword"
                @on-enter="handleSearch"
              />
            </MenuItem>
          </div>
        </Menu>
      </Header>
      <div class="genderSelect">
        <a>
          <span @click="handleSelect('all')">
            <Icon type="md-transgender" /> All
          </span>
        </a>
        <a @click="handleSelect('male')"
          ><span><Icon type="md-male" /> Male </span>
        </a>
        <a>
          <span @click="handleSelect('female')">
            <Icon type="md-female" /> Female
          </span>
        </a>
      </div>
      <Content
        :style="{
          margin: '10px 20px',
          background: '#fff',
          minHeight: '565px',
        }"
      >
        <div class="content">
          <div v-for="list in displayLists" :key="list.id">
            <div class="card" :key="list.id" @click="showModal(list)">
              <Card style="width: 200px">
                <Avatar :src="list.avatar" size="150" />
                <h3>{{ list.name }}</h3>
              </Card>
            </div>
          </div>
        </div>
      </Content>
      <Footer class="layout-footer-center">
        <Page
          :total="getTotalPages"
          @on-change="pageChange"
          :current="currentPage"
        />
      </Footer>
    </Layout>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      base_URL: "https://lighthouse-user-api.herokuapp.com/api/v1/users/",
      lists: [],
      maleLists: [],
      femaleLists: [],
      filteredLists: [],
      eachPageLists: [],
      keyword: "",
      gender: "all",
      currentPage: 1,
      totalPages: null,
      item_per_page: 10,
    };
  },
  mounted() {
    axios
      .get(this.base_URL)
      .then((res) => {
        this.lists.push(...res.data.results);
        this.maleLists = this.lists.filter((user) => user.gender !== "female");
        this.femaleLists = this.lists.filter(
          (user) => user.gender === "female"
        );
      })
      .catch((error) => {
        console.log(error);
      });
  },
  methods: {
    showModal(data) {
      this.$Modal.success({
        title: data.name,
        content: `
        <p>age: ${data.age}</p>
        <p>Birthday: ${data.birthday}</p>
        <p>Region: ${data.region}</p>
        <p>Email: ${data.email}</p>
        <p>Gender: ${data.gender}</p>`,
        okText: "Confirm",
      });
    },
    // 換頁
    pageChange(page) {
      this.currentPage = page;
    },
    // 搜尋
    handleSearch() {
      const regex = new RegExp(this.keyword, "i");
      switch (this.gender) {
        case "all":
          this.filteredLists = this.lists.filter(
            (user) => user.surname.match(regex) || user.name.match(regex)
          );
          this.keyword = "";
          break;
        case "male":
          this.filteredLists = this.maleLists.filter(
            (user) => user.surname.match(regex) || user.name.match(regex)
          );
          this.keyword = "";
          break;
        case "female":
          this.filteredLists = this.femaleLists.filter(
            (user) => user.surname.match(regex) || user.name.match(regex)
          );
          this.keyword = "";
          break;
      }
    },
    handleSelect(gender) {
      this.currentPage = 1;
      this.gender = gender;
      this.handleSearch();
    },
  },
  computed: {
    // 渲染頁面
    displayLists() {
      let listsRange = (this.currentPage - 1) * this.item_per_page;
      if (this.filteredLists.length !== 0) {
        return (this.eachPageLists = this.filteredLists.slice(
          listsRange,
          listsRange + this.item_per_page
        ));
      }
      return (this.eachPageLists = this.lists.slice(
        listsRange,
        listsRange + this.item_per_page
      ));
    },
    // 取得總頁數
    getTotalPages() {
      if (this.filteredLists.length !== 0) {
        return this.filteredLists.length;
      }
      return this.lists.length;
    },
  },
};
</script>

<style lang='scss'>
.content {
  display: flex;
  flex-flow: row wrap;
  justify-content: space-around;
  align-items: center;
  // border: 1px solid black;
  height: auto;
  // width: 80%;
  .card {
    margin: 20px 20px;
    h3 {
      margin-top: 10px;
    }
  }
}
.nav {
  z-index: 1;
}
.genderSelect {
  display: flex;
  flex-flow: row nowrap;
  justify-content: center;
  align-items: center;
  margin-top: 10vh;
  a {
    margin: 0 20px;
  }
}
</style>