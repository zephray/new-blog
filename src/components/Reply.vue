<template lang="pug">
  div.reply: div.card
    h3.title 评论
    div.content
      ul.replies-list(v-if="replies && replies.length !== 0")
        li(v-for="reply in replies")
          div.name {{ reply.user }} {{ reply.site !== '' ? `from ${reply.site}` : ''}}
          div.date {{ timeToString(reply.datetime) }}
          div(v-html="reply.content" v-if="reply.markdown")
          div.raw-content(v-else) {{ reply.content }}
      div.send-new
        h3 发表评论
        table.form-table: tbody
          tr
            th 姓名
            td: input.full(v-model="name")
          tr
            th 站点
            td: input.full(v-model="site")
          tr
            th 电子邮件
            td: input.full(v-model="email", placeholder="邮件地址不会公开")
          tr
            th 评论
            td: textarea.content(v-model="content", placeholder="We support markdown!")
          tr
            td
            td: button(@click="submit") SUBMIT
</template>

<script>
import timeToString from '../utils/timeToString';
import api from '../api';

export default {
  name: 'Reply',
  props: {
    replies: {
      type: Array,
      required: true,
    },
    'api-path': {
      type: String,
      required: true,
    },
    'refresh-replies': {
      type: Function,
      required: true,
    }
  },
  data () {
    return {
      name: '',
      email: '',
      content: '',
      site: '',
    };
  },
  watch: {
    replies () {
      this.name = this.email = this.content = this.site = '';
    }
  },
  methods: {
    timeToString,
    submit () {
      let data = {
        user: this.name,
        email: this.email,
        site: this.site,
        content: this.content,
      };
      if (!data.user) {
        alert('姓名是必填项呢');
        return;
      } else if (!data.content) {
        alert('评论内容为空呢');
        return;
      }
      api[this.$props.apiPath].putReplyBySlug({ slug: this.$route.params.slug, data })
        .then(() => {
          if (this.$store.state.forceReload) {
            window.location.href = window.location.href;
          } else {
            this.$props.refreshReplies();
          }
        });
    }
  },
};
</script>

<style lang="scss">
@import '../style/global.scss';
@import '../style/form-table.scss';

// label.md {
//   height: 0;
//   overflow: show;
//   pointer-events: none;
//   display: block;
//   transform: translateY(-50px);
//   font-size: 12px;
//   transition: all linear 0.1s;
//   color: grey;
// }

// input.md:not(:focus):invalid + label.md {
//   font-size: initial;
//   transform: translateY(-30px);
// }

// input.md {
//   width: 200px;
//   margin-top: 16px;
//   padding-top: 8px;
//   padding-bottom: 8px;
//   border: none;
//   border-bottom: 2px solid $button_color;
// }

// input.md:focus {
//   outline: none;
// }

div.reply {
  div.content {
    padding: 0.2em 1em 0.2em 1em;
  }
  table th {
    text-align: right;
    width: 90px;
    vertical-align: top;
    font-weight: normal;
  }
  table {
    width: 90%;
    border-spacing: 5px;
  }
  table textarea.content {
    height: 8em;
  }

  div.raw-content {
    white-space: pre;
  }

  ul.replies-list {
    padding: 0;
    list-style: none;

    > li {
      padding: 0.6em 1em 0.1em 1em;
      margin: 0.5em 0 0.5em 0;
      background-color: rgb(245, 245, 245);
      border-radius: 2px;
      line-height: 1.2em;

      pre {
        background-color: inherit;
        border: rgb(235, 235, 235);
        border-radius: 0;
      }
    }

    div.name {
      font-weight: bold;
    }
    div.date, div.name {
      font-size: 0.8em;
      color: grey;
    }
  }
}
</style>