<template>
    <div>
        <GistAuthModal
            v-if="isLoginModalOpen"
            @close-modal="isLoginModalOpen = false"
        />
        <a
            @click="toggleStar"
            class="flex flex-row justify-center mb-3 text-sm rounded shadow cursor-pointer text-grey-darker w-100 sm:mb-auto sm:bg-white"
            :class="{
                'opacity-100': isLoggedIn,
                'opacity-50': !isLoggedIn,
            }"
        >
        <div class="inline-block flex flex-row justify-center px-2 py-3 sm:py-1">
            <svg
                :class="{
                    'text-blue-dark': isStarred,
                    'text-grey-light': !isStarred,
                }"
                class="w-4 h-5 mr-1 fill-current"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
            >
                <path
                    d="M10 15l-5.878 3.09 1.123-6.545L.489 6.91l6.572-.955L10 0l2.939 5.955 6.572.955-4.756 4.635 1.123 6.545z"
                />
            </svg>
            <span v-if="isStarred">Unstar</span>
            <span v-else>Star</span>
        </div>
        <div v-if="isLoggedIn" class="inline-block px-2 py-3 sm:py-1 border-l border-grey-lighter">{{totalStars}}</div>
        </a>
    </div>
</template>

<script>
export default {
    props: {
        isStarredForUser: Boolean,
        gistId: String,
        isLoggedIn: Boolean,
    },
    data: function () {
        return {
            isStarred: !!this.isStarredForUser,
            starRoute: route('post.star', { gistId: this.gistId }),
            unstarRoute: route('post.unstar', { gistId: this.gistId }),
            isLoginModalOpen: false,
            stargazerRoute: route('post.starcount', { gistId: this.gistId }),
            totalStars: ""
        };
    },
    mounted() {
        if(this.isLoggedIn) {
            this.starCount()
        }
    },
    methods: {
        toggleStar() {
            if (!this.isLoggedIn) {
                this.isLoginModalOpen = true
                return
            }

            axios({
                withCredentials: true,
                method: this.isStarred ? 'delete' : 'put',
                url: this.toggleRoute(),
            }).then((response) =>
                this.isStarred
                    ? (this.isStarred = false, this.totalStars--)
                    : (this.isStarred = true, this.totalStars++)
            );
        },
        toggleRoute() {
            return this.isStarred ? this.unstarRoute : this.starRoute;
        },
        async starCount() {
            const response = await fetch(this.stargazerRoute)
            const res = await response.json()
            this.totalStars = await res.data.viewer.gist.stargazerCount 
        }
    },
};
</script>
