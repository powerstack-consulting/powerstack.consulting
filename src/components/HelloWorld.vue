<template lang='pug'>
  .app
    article(v-if='gists.length' v-for='gist in gists')
      h2 {{gist.title}}
      vue-markdown {{gist.content}}
      small by {{gist.author}} on {{gist.date}}
    
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import VueMarkdown from 'vue-markdown';

@Component({
  components: {
    VueMarkdown,
  },
})
export default class HelloWorld extends Vue {

  public gists = [];
  @Prop() private msg!: string;

  public async mounted() {
    await this.fetchData();
  }

  public async fetchData() {
    const BASE_URL = 'https://api.github.com';
    const axios = (this as any).$axios;
    const gistList = await axios.get(
      `${BASE_URL}/users/powerstack-consulting/gists`,
    );
    const gistPromises = gistList.data.map((gist: { id: string }) => {
      return axios.get(`${BASE_URL}/gists/${gist.id}`);
    });
    const resp = await Promise.all(gistPromises);
    (this as any).gists = resp.map((item: any) => {
      const meta = JSON.parse(item.data.files['meta.json'].content);
      const content = item.data.files['content.md'].content;
      return {
        content,
        title: meta.title,
        author: meta.author,
        date: meta.date,
      };
    });
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
