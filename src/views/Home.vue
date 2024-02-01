<template>
    <audio autoplay loop volume="0.1" id="bg-song">
        <source src="/audio/flower_ost_peaceful_repose.mp3" type="audio/mpeg">
    </audio>
    <div class="p-2 flex flex-col items-center justify-center h-screen overflow-hidden relative">
        <div class="absolute top-0 left-0 w-[100vw] h-[100vh] pointer-events-none" ref="starsContainer">

        </div>
        <div id="question-container" class="flex flex-col items-center gap-6 anim-renew" ref="questionContainer">
            <h1 class="text-3xl text-center max-w-[50vw] leading-snug">{{ questionList[questionIdx].question }}</h1>
            <div class="flex flex-row gap-3" v-if="questionList[questionIdx].answers">
                <button
                    v-for="ans, k in questionList[questionIdx].answers"
                    :key="k"
                    @click="answer(ans)"
                    class="void-button"
                >{{ ans }}</button>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

const questionContainer = ref<HTMLElement | null>(null);
const starsContainer = ref<HTMLElement | null>(null);
const questionIdx = ref(0);
const answers = ref<string[]>([]);
const NUM_STARS = 200;
const SCALE_PER_DEPTH = 0.5;
const MAX_DEPTH = 4;
const STAR_ANIM_DURATION = 3000;
const STAR_COLORS = ['#ffd27d', '#ffa371', '#a6a8ff', '#fffa86', '#a87bff'];
const GOODBYE_DURATION = 4000;
const ROT_SHIFT_DEGREES = 30;
const ROT_ANIM_DURATION = 3000;
let flagEnding = false;

onMounted(() => {
    generateStarAnims();
    generateDynaRot();
    addStars();
    // Set CSS root variable
    document.documentElement.style.setProperty('--star-animation-duration', `${STAR_ANIM_DURATION}ms`);
    document.documentElement.style.setProperty('--goodbye-duration', `${GOODBYE_DURATION}ms`);
    questionContainer.value!.style.animationDuration = `${STAR_ANIM_DURATION}ms`;

    // Fix the initial timing misalignment
    setTimeout(() => {
        questionContainer.value?.style.removeProperty('animation-duration');
    }, STAR_ANIM_DURATION * 2);
})

function addStars() {
    const starSystem = document.createElement('div');
    starSystem.classList.add('absolute', 'top-0', 'left-0', 'w-full', 'h-full', 'pointer-events-none');
    for (let i = 0; i < NUM_STARS; i++) {
        const star = document.createElement('div');
        const starSize = randInt(2, 4);
        star.classList.add('absolute', 'rounded-full', 'transition', 'duration-200', 'blur-[0px]');
        star.style.backgroundColor = STAR_COLORS[randInt(0, STAR_COLORS.length - 1)];
        star.style.width = `${starSize}px`;
        star.style.height = `${starSize}px`;
        star.style.left = `${randInt(-20, 120)}%`;
        star.style.top = `${randInt(-50, 150)}%`;
        star.style.animationDelay = `${randInt(0, 1000)}ms`;
        star.style.animationDuration = `${randInt(2000, 3000)}ms`;
        star.style.animationIterationCount = 'infinite';
        star.style.animationName = 'flicker';
        // Glow
        star.style.boxShadow = `0 0 ${starSize * 1}px ${starSize * 0.5}px ${star.style.backgroundColor}`;

        // star.style.animationDuration = `${Math.random() * 5 + 5}s`;
        // star.style.animationDelay = `${Math.random() * 5}s`;
        starSystem.appendChild(star);
    }
    starSystem.dataset.depth = '0';
    starSystem.classList.add('anim-stars-intro');
    starsContainer.value?.appendChild(starSystem);
}

function pushbackStars() {
    const starSystems = starsContainer.value?.querySelectorAll('div');
    starSystems?.forEach(starSystem => {
        const depth = parseInt(starSystem.dataset.depth ?? '0');
        if (depth < MAX_DEPTH) {
            starSystem.dataset.depth = `${depth + 1}`;
            // starSystem.style.transform = `scale(${SCALE_PER_DEPTH ** (depth + 1)})`;
            if (starSystem.classList.contains('anim-stars-intro')) {
                starSystem.classList.remove('anim-stars-intro');
            }
            starSystem.classList.remove(`anim-stars-${depth-1}`);
            starSystem.classList.add(`anim-stars-${depth}`);
            if (depth === MAX_DEPTH - 1) {
                setTimeout(() => {
                    starSystem.remove();
                }, STAR_ANIM_DURATION);
            }
        }
    })
}

function generateStarAnims() {
    // Create a style element
    const dynAnim = document.createElement('style');
    // Add the animations to the style element
    for (let i = 0; i < MAX_DEPTH; i++) {
        const animName = `stars-${i}`;
        const anim = `
            @keyframes ${animName} {
                0% {
                    transform: scale(${SCALE_PER_DEPTH ** i});
                }
                100% {
                    transform: scale(${SCALE_PER_DEPTH ** (i + 1)});
                    ${i === MAX_DEPTH - 1 ? 'opacity: 0;' : ''}
                }
            }

            .anim-${animName} {
                /* Do not reset animation on class removal */
                animation-fill-mode: forwards;
                animation-name: ${animName};
                animation-duration: ${STAR_ANIM_DURATION}ms;
                animation-timing-function: ease-in-out;
                /* animation-timing-function: linear; */
            }
        `;
        dynAnim.appendChild(document.createTextNode(anim));
    }

    // Add the default animation
    const anim = `
        @keyframes stars-intro {
            0% {
                transform: scale(5);
                opacity: 0;
            }
            60% {
                opacity: 1;
            }
            100% {
                transform: scale(1);
            }
        }

        .anim-stars-intro {
            animation-fill-mode: forwards;
            animation-name: stars-intro;
            animation-duration: ${STAR_ANIM_DURATION}ms;
            animation-timing-function: ease-in-out;
        }
    `;
    dynAnim.appendChild(document.createTextNode(anim));

    // Add the flicker animation
    const flickerAnim = `
        @keyframes flicker {
            0% {
                opacity: 0.5;
            }
            50% {
                opacity: 1;
            }
            100% {
                opacity: 0.5;
            }
        }
    `;
    dynAnim.appendChild(document.createTextNode(flickerAnim));

    // Add the style element to the document
    document.head.appendChild(dynAnim);
}

function generateDynaRot() {
    const maxAnims = 360 / ROT_SHIFT_DEGREES;
    const dynAnim = document.createElement('style');
    for (let i = 0; i < maxAnims; i++) {
        const animName = `dyna-rot-${i}`;
        const anim = `
            @keyframes ${animName} {
                0% {
                    transform: rotate(${i * ROT_SHIFT_DEGREES}deg);
                }
                100% {
                    transform: rotate(${(i + 1) * ROT_SHIFT_DEGREES}deg);
                }
            }

            .anim-${animName} {
                /* Do not reset animation on class removal */
                animation-fill-mode: forwards;
                animation-name: ${animName};
                animation-duration: ${ROT_ANIM_DURATION}ms;
                animation-timing-function: ease-in-out;
            }
        `;
        dynAnim.appendChild(document.createTextNode(anim));
    }
    document.head.appendChild(dynAnim);

}

function doGalaxyRotation() {
    let level = parseInt(questionContainer.value.dataset.rotaLevel ?? '0');
    const maxAnims = 360 / ROT_SHIFT_DEGREES;
    if (level >= maxAnims) {
        level = 0;
    }
    const animName = `dyna-rot-${level}`;
    // Remove any class that starts with anim-dyna-rot
    const classes = Array.from(starsContainer.value.classList).filter(c => c.startsWith('anim-dyna-rot'));
    classes.forEach(c => {
        starsContainer.value.classList.remove(c);
    })

    // Add the new class
    starsContainer.value.classList.add(`anim-${animName}`);
    console.log(animName);

    // Increment the level and set the data attribute
    level++;
    questionContainer.value.dataset.rotaLevel = `${level}`;
}

function answer(ans: string) {
    const audioNode = document.getElementById('bg-song') as HTMLAudioElement;
    if (audioNode.paused) {
        audioNode.play();
    }
    answers.value.push(ans);
    if (questionContainer.value?.classList.contains('anim-renew')) {
        questionContainer.value?.classList.remove('anim-renew');
    }
    questionContainer.value?.classList.add('anim-dissonance');
    pushbackStars();
    addStars();
    doGalaxyRotation();
    setTimeout(() => {
        // If we haven't reached the ending yet, go to the next question
        if (!flagEnding) {
            questionIdx.value++;
        }
        // Ending
        if (questionIdx.value >= questionList.length - 1 && !flagEnding) {
            flagEnding = true;
            setTimeout(() => {
                questionContainer.value?.classList.add('anim-goodbye');
                setTimeout(() => {
                    questionContainer.value?.classList.add('hidden');
                }, GOODBYE_DURATION);
                setInterval(() => {
                    answer('');
                }, STAR_ANIM_DURATION);
            }, 2000);
        }
        questionContainer.value?.classList.remove('anim-dissonance')
        questionContainer.value?.classList.add('anim-renew')
        setTimeout(() => {
            questionContainer.value?.classList.remove('anim-renew')
        }, STAR_ANIM_DURATION / 2)
    }, STAR_ANIM_DURATION / 2)
}

function randInt(min: number, max: number) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}

const questionList = [
    {
        question: 'Hey you. Yeah, the one reading this.',
        answers: [
            'Me?',
            'Who, me?',
            'What?',
        ]
    },
    {
        question: 'Yes, you. I have a question for you.',
        answers: [
            'What is it?',
            'What?',
            'Go ahead.',
            'Shoot.',
        ]
    },
    {
        question: 'Take a look around, what do you see?',
        answers: [
            'I see stars',
            'I see the sky',
            'I see space'
        ]
    },
    {
        question: 'Why yes indeed. For a brief moment, you are in space. The vastness of the universe is all around you.\
        The sky is infinite, all of it filled to the brim with stars.',
        answers: [
            'What are stars like?',
        ],
    },
    {
        question: 'All stars are different, some are bigger, some are smaller. Some are brighter, some are dimmer.',
        answers: [
            '...',
        ],
    },
    {
        question: 'For every star you see, there are millions more that you don\'t. Just in these few moments\
        you\'ve been here, you\'ve seen more stars than you can count.',
        answers: [
            '...',
        ],
    },
    {
        question: 'Much like humans, stars are born, they live, and they die.',
        answers: [
             'What happens when a star dies?',
        ],
    },
    {
        question: 'When a star dies, it explodes in a brilliant supernova. The star\'s remains are scattered across\
        the universe, and from them, new stars are born.',
        answers: [
            'So they go out with a bang?',
        ],
    },
    {
        question: 'They sure do. And when they\'re alive, they shine bright.',
        answers: [
            'And that is to say...?',
        ],
    },
    {
        question: 'That is to say, you too can shine bright. You too can be a star. In a way, we\'re all stars.',
        answers: [
            '...',
        ],
    },
    {
        question: 'And much like opportunities, stars are everywhere. You just have to look for them.',
        answers: [
            'What do you mean?',
        ],
    },
    {
        question: 'All you have to do is reach out and grab it. It\'s that simple. Go for a hike, pick up a book, start a new hobby.\
        Talk to that person you\'ve been meaning to talk to. Do it today, and do it now. Stars live much longer than humans, so use\
        your time wisely.',
        answers: [
            'I\'ll try.',
            'I will.',
            'I\'ll consider it.',
        ],
    },
    {
        question: 'Thank you for your time. Now, enjoy the stars for however long you\'d like.',
    }
]
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Indie+Flower&display=swap');

body {
    font-family: "Indie Flower", cursive;
    font-weight: 400;
    font-style: normal;
}

:root {
    --star-animation-duration: 0s;
    --goodbye-duration: 0s;
}

.void-button {
    background-color: #000000;
    color: #ffffff;
    border: 1px solid #ffffff;
    border-radius: 5px;
    padding: 5px 10px;
    transition: all 0.3s ease-in-out;
}

.void-button:hover {
    background-color: #ffffff;
    color: #000000;
    scale: 1.1;
}

.void-button:active {
    background-color: #ffffff;
    color: #000000;
}

@keyframes dissonance {
    0% {
        /* transform: translateY(0%); */
    }
    100% {
        /* transform: translateY(-200%); */
        scale: 0.5;
        opacity: 0;
        filter: blur(10px) brightness(0.5)
    }
    
}

.anim-dissonance {
    /* Do not reset animation on class removal */
    animation-fill-mode: forwards;
    animation-name: dissonance;
    animation-duration: calc(var(--star-animation-duration) / 2);
    animation-timing-function: ease-in-out;
    pointer-events: none
}

@keyframes renew {
    0% {
        /*transform: translateY(200%)*/;
        scale: 2.5;
        opacity: 0;
        filter: blur(10px) brightness(0.5)
    }
    100% {
        transform: translateY(0%);
    }
}

.anim-renew {
    /* Do not reset animation on class removal */
    animation-fill-mode: forwards;
    animation-name: renew;
    animation-duration: calc(var(--star-animation-duration) / 2);
    animation-timing-function: ease-in-out;
}

@keyframes rotate-sky {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
};

.anim-rotate-sky {
    animation-name: rotate-sky;
    animation-duration: 30s;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
}

@keyframes goodbye {
    0% {
        opacity: 1;
    }
    100% {
        opacity: 0;
    }
}

.anim-goodbye {
    animation-name: goodbye;
    animation-duration: var(--goodbye-duration);
    animation-timing-function: ease-in-out;
    animation-iteration-count: 1;
    animation-fill-mode: forwards;
}
</style>