<template>
    <div
        class="theme-container"
        :class="pageClasses"
        @touchstart="onTouchStart"
        @touchend="onTouchEnd"
    >
        <ClientOnly>
            <Navigation></Navigation>
        </ClientOnly>
        <!-- <div class="empty"></div> -->
		<ClientOnly>
			<div class="main_content_wrapper">
				<Home v-if="$page.frontmatter.home"></Home>
				<NewHome v-if="$page.frontmatter.isNewHome"></NewHome>
				<Developer v-if="$page.frontmatter.isDeveloper"></Developer>
					<Community v-if="$page.frontmatter.isCommunity"/>
					<div class="md_container" v-if="showMd">
						<div class="md_wrap">
							<Markdown  :articleDetails='$store.state.articleData'></Markdown>
							<Content :slot-key="$store.state.articleData ? $store.state.articleData.slot :''"></Content>
						</div>
					</div>
			</div>
		</ClientOnly>
        <Footer></Footer>
    </div>
</template>

<script>
import Home from '@theme/components/Home.vue'
import Navbar from '@theme/components/Navbar.vue'
import Page from '@theme/components/Page.vue'
import Sidebar from '@theme/components/Sidebar.vue'
import { resolveSidebarItems } from '../util'
import Navigation from "@theme/components/Navigation.vue";
import Footer from "@theme/components/Footer.vue";
import NewHome from "@theme/components/NewHome.vue";
import Developer from "../components/Developer";
import Community from "../components/Community";
import Markdown from "../components/Markdown"
const nav = require('../../config.js');

export default {
    name : 'Layout',
    data(){
        return {
            isSidebarOpen : false,
            blogAndArticleList:[],
			articleDetails: null,
			articleName:null,
			articleData:null,
        }
    },
    components : {
        Home,
        Page,
        Sidebar,
        Navbar,
        Navigation,
        Footer,
        NewHome,
        Developer,
        Community,
        Markdown
    },
    watch:{
        frontmatter(frontmatter){
            if(frontmatter.isCommunity){
                this.setBlogAndArticleList(frontmatter)
            }
        },
		'$store.state.articleData'(){
		
		},
        $route:{
            handler(val,oldval){
                nav.themeConfig.nav.forEach((item,index)=>{
                    if(item.link === val.path) {
                        this.$store.commit('currentIndex',index);
                    }
                })
                // console.log(val);//新路由信息
                // console.log(oldval);//老路由信息
            },
            immediate: true,
            // 深度观察监听
            deep: true
        }
    },
	
    mounted(){
		
        // console.log(nav)
        if(localStorage.getItem('currentIndex')) {
            this.$store.commit('currentIndex',JSON.parse(localStorage.getItem('currentIndex')))
        }
        if(JSON.parse(sessionStorage.getItem('article'))){
			this.$store.commit('articleData',JSON.parse(sessionStorage.getItem('article')))
		}
        this.$router.afterEach(() => {
            this.isSidebarOpen = false
            window.scrollTo(0, 0)
        })
        if(this.$page.frontmatter.isCommunity){
            this.setBlogAndArticleList(this.$page.frontmatter)
        }
        window.onhashchange = function (e) {
            console.log(e);
            console.log(e.newURL);
            console.log(e.oldURL);
        }
    },
    computed : {
    
		/*articleDetails(){
			return  JSON.parse(sessionStorage.getItem('article'));
		},*/
        showMd(){
            return Object.keys(this.$page.frontmatter).length === 0;
        },
        frontmatter(){
           return this.$page.frontmatter
        },
        shouldShowNavbar(){
            const {themeConfig} = this.$site
            const {frontmatter} = this.$page
            if(
                frontmatter.navbar === false
                || themeConfig.navbar === false){
                return false
            }
            return (
                this.$title
                || themeConfig.logo
                || themeConfig.repo
                || themeConfig.nav
                || this.$themeLocaleConfig.nav
            )
        },

        shouldShowSidebar(){
            const {frontmatter} = this.$page
            return (
                !frontmatter.home
                && frontmatter.sidebar !== false
                && this.sidebarItems.length
            )
        },

        sidebarItems(){
            return resolveSidebarItems(
                this.$page,
                this.$page.regularPath,
                this.$site,
                this.$localePath
            )
        },

        pageClasses(){
            const userPageClass = this.$page.frontmatter.pageClass
            return [
                {
                    'no-navbar' : !this.shouldShowNavbar,
                    'sidebar-open' : this.isSidebarOpen,
                    'no-sidebar' : !this.shouldShowSidebar
                },
                userPageClass
            ]
        }
    },
    methods : {
        toggleSidebar(to){
            this.isSidebarOpen = typeof to === 'boolean' ? to : !this.isSidebarOpen
            this.$emit('toggle-sidebar', this.isSidebarOpen)
        },
        setBlogAndArticleList(frontmatter){
            let articles = JSON.parse(JSON.stringify(frontmatter.articles));
            let blogs = JSON.parse(JSON.stringify(frontmatter.blogs));
            this.blogAndArticleList = [...articles, ...blogs];
        },
        // side swipe
        onTouchStart(e){
            this.touchStart = {
                x : e.changedTouches[0].clientX,
                y : e.changedTouches[0].clientY
            }
        },

        onTouchEnd(e){
            // const dx = e.changedTouches[0].clientX - this.touchStart.x
            // const dy = e.changedTouches[0].clientY - this.touchStart.y
            // if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) > 40) {
            //   if (dx > 0 && this.touchStart.x <= 80) {
            //     this.toggleSidebar(true)
            //   } else {
            //     this.toggleSidebar(false)
            //   }
            // }
        }
    }
}
</script>
<style lang="stylus">
@import '../styles/adaptation.styl';
.theme-container{
    display flex
    flex-direction: column;
    //padding-top $navbarHeight
    width 100%
    height 100%

    .main_content_wrapper{
        // flex 1
        // -webkit-box-flex: 1;
        // -moz-box-flex: 1;
        // -webkit-flex: 1;
        // -ms-flex: none; 
        margin-top: 6.4rem;
        .md_container{
            font-size: 15px;
            font-family: PingFangSC-Medium, PingFang SC;
            font-weight: 400;
            color: rgba(0,0,0,0.75);
            line-height: 36px;
            //display:flex;
            .md_wrap{
                padding-top:60px;
                padding-bottom:80px;
				@media (max-width 1200px){
                    padding-left: 48px;
                    padding-right: 48px;
				}
                @media (max-width: 768px){
                    padding-left: 24px;
                    padding-right: 24px;
                }
                // h2, h3{
                //     border-bottom: none;
                // }
                img {
                    width: 100%;
                }
            }

        }
    }
}



@media screen and (min-width: $minPcWidth) {
    .theme-container{
        .main_content_wrapper{

            display:flex;
            justify-content center;
            .md_container{
                width:960px;
                //display:flex;
                .md_wrap{

                }

            }
        }
    }

}
</style>
