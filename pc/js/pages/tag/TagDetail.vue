<template>
    <div id="tag-entry-list-box">
        <header class="list-header">
            <div class="left">
                <img :src="tag.iconURL" class="tag-icon">
                <button class="subscribe-btn" :class="{subscribed: isFollowed}"
                    @click.stop.prevent="onFollow" @mouseenter="onMouseenter" @mouseleave="onMouseleave">{{followText}}</button>
            </div>
            <ul class="nav-list">
                <li class="nav-item" :class="{active: query.sort === 'newest'}">
                    <a @click="onSort('newest')">最新</a>
                </li>
                <li class="nav-item" :class="{active: query.sort === 'popular'}">
                    <a @click="onSort('popular')">最热</a>
                </li>
            </ul>
        </header>
        <div>
            <Pinterest ref="articlePinterest" :url="url" :query="query" @load="onLoad">
                <template v-slot:loading>
                    <div style="padding: 20px; padding-top: 10px;">
                        <ArticleLoading />
                    </div>
                </template>
                <template v-slot:content>
                    <div>
                        <div class="article-list">
                            <ArticleItem :key="article.id" v-for="article in articles" :article="article" />
                        </div>
                    </div>
                </template>
            </Pinterest>
        </div>
    </div>
</template>

<script>
import ArticleLoading from '~/js/components/article/ArticleLoading.vue';
import ArticleItem from '~/js/components/article/ArticleItem.vue';
import { myHTTP } from '~/js/common/net.js';
import { ErrorCode } from '~/js/constants/error.js';
import Pinterest from '~/js/components/common/Pinterest.vue';

export default {
    data () {
        return {
            isFollowed: window.isFollowed,
            isMouseEnter: false,
            tag: window.tag,
            articles: [],
            url: `/tags/${window.tag.id}/articles`,
            query: {
                sort: 'newest',
            }
        };
    },
    computed: {
        followText() {
            if (this.isFollowed && this.isMouseEnter) {
                return '取消关注';
            }
            if (this.isFollowed) {
                return '已关注';
            }
            return '关注';
        }
    },
    methods: {
        onLoad(result) {
            this.articles = this.articles.concat(result.data.data.list);
        },
        onSort(sort) {
            if (sort === this.query.sort) {
                return;
            }
            this.articles = [];
            this.query = {
                ...this.query,
                sort,
            };
            this.$refs.articlePinterest.refresh(this.query);
        },
        onMouseenter() {
            this.isMouseEnter = true;
        },
        onMouseleave() {
            this.isMouseEnter = false;
        },
        onFollow () {
            const url = `/tags/${this.tag.id}/follow`;
            let reqMethod;
            if (this.isFollowed) {
                reqMethod = myHTTP.delete;
            } else {
                reqMethod = myHTTP.post;
            }
            reqMethod(url).then((res) => {
                if (res.data.errorCode === ErrorCode.SUCCESS.CODE) {
                    this.isFollowed = !this.isFollowed;
                    if (this.isFollowed) {
                        tag.followerCount++;
                    } else {
                        tag.followerCount--;
                    }
                    document.getElementById('followerAndArticleCount').innerHTML = `${tag.followerCount} 关注，${tag.articleCount} 文章`;
                }
            });
        },
    },
    components: {
        ArticleLoading,
        ArticleItem,
        Pinterest,
    }
}
</script>

