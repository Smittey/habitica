<template>
  <div>
    <yesterdaily-modal
      :yester-dailies="yesterDailies"
      @run-cron="runYesterDailiesAction()"
    />
    <armoire-empty />
    <new-stuff />
    <death />
    <low-health />
    <level-up />
    <choose-class />
    <testing />
    <testingletiant />
    <rebirth-enabled />
    <drops-enabled />
    <contributor />
    <won-challenge />
    <ultimate-gear />
    <streak />
    <rebirth />
    <joined-guild />
    <joined-challenge />
    <invited-friend />
    <login-incentives :data="notificationData" />
    <quest-completed />
    <quest-invitation />
    <verify-username />
    <generic-achievement :data="notificationData" />
    <just-add-water />
    <lost-masterclasser />
    <mind-over-matter />
  </div>
</template>

<style lang='scss'>
  .introjs-helperNumberLayer, .introjs-bullets {
    display: none;
  }

  .introjs-tooltip {
    background-image: url('~@/assets/svg/for-css/tutorial-border.svg');
    background-size: 100% 100%;
    background-repeat: no-repeat;
    height: 131px;
    min-height: 131px !important;
    width: 400px;
    max-width: 400px;

    .featured-label {
      position: absolute;
      top: -1.5em;
    }

    .npc_justin_textbox {
      position: absolute;
      right: 1em;
      top: -3.6em;
      width: 48px;
      height: 52px;
      background-image: url('~@/assets/images/justin_textbox.png');
    }
  }

  .introjs-button:hover, .introjs-button:active, .introjs-button:focus {
    background-image: none;
    background-color: #4f2a93 !important;
    color: #fff;
  }

  .introjs-tooltipbuttons {
    margin-top: 1em;
  }

  .introjs-button {
    text-shadow: none;
    text-align: center;
    display: block;
    max-width: 90px;
    font-family: Roboto;
    font-size: 14px;
    background-image: none;
    color: #fff;
    margin: 0 auto;
    padding: .8em;
    border-radius: 2px;
    background-color: #4f2a93 !important;
    box-shadow: 0 2px 2px 0 rgba(26, 24, 29, 0.16), 0 1px 4px 0 rgba(26, 24, 29, 0.12) !important;
  }
</style>

<script>
import axios from 'axios';
import moment from 'moment';
import throttle from 'lodash/throttle';
import debounce from 'lodash/debounce';

import { toNextLevel } from '@/../../common/script/statHelpers';
import { shouldDo } from '@/../../common/script/cron';
import { mapState } from '@/libs/store';
import notifications from '@/mixins/notifications';
import guide from '@/mixins/guide';

import yesterdailyModal from './yesterdailyModal';
import newStuff from './achievements/newStuff';
import death from './achievements/death';
import lowHealth from './achievements/lowHealth';
import levelUp from './achievements/levelUp';
import chooseClass from './achievements/chooseClass';
import armoireEmpty from './achievements/armoireEmpty';
import questCompleted from './achievements/questCompleted';
import questInvitation from './achievements/questInvitation';
import testing from './achievements/testing';
import testingletiant from './achievements/testingletiant';
import rebirthEnabled from './achievements/rebirthEnabled';
import dropsEnabled from './achievements/dropsEnabled';
import contributor from './achievements/contributor';
import invitedFriend from './achievements/invitedFriend';
import joinedChallenge from './achievements/joinedChallenge';
import joinedGuild from './achievements/joinedGuild';
import rebirth from './achievements/rebirth';
import streak from './achievements/streak';
import ultimateGear from './achievements/ultimateGear';
import wonChallenge from './achievements/wonChallenge';
import genericAchievement from './achievements/genericAchievement';
import justAddWater from './achievements/justAddWater';
import lostMasterclasser from './achievements/lostMasterclasser';
import mindOverMatter from './achievements/mindOverMatter';
import loginIncentives from './achievements/login-incentives';
import verifyUsername from './settings/verifyUsername';

const NOTIFICATIONS = {
  CHALLENGE_JOINED_ACHIEVEMENT: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('joinedChallenge')}`,
    modalId: 'joined-challenge',
  },
  ULTIMATE_GEAR_ACHIEVEMENT: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('gearAchievementNotification')}`,
    modalId: 'ultimate-gear',
  },
  GUILD_JOINED_ACHIEVEMENT: {
    label: $t => `${$t('achievement')}: ${$t('joinedGuild')}`,
    achievement: true,
    modalId: 'joined-guild',
  },
  INVITED_FRIEND_ACHIEVEMENT: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('invitedFriend')}`,
    modalId: 'invited-friend',
  },
  NEW_CONTRIBUTOR_LEVEL: {
    achievement: true,
    label: $t => $t('modalContribAchievement'),
    modalId: 'contributor',
  },
  ACHIEVEMENT_ALL_YOUR_BASE: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('achievementAllYourBase')}`,
    modalId: 'generic-achievement',
  },
  ACHIEVEMENT_BACK_TO_BASICS: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('achievementBackToBasics')}`,
    modalId: 'generic-achievement',
  },
  ACHIEVEMENT_DUST_DEVIL: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('achievementDustDevil')}`,
    modalId: 'generic-achievement',
  },
  ACHIEVEMENT_ARID_AUTHORITY: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('achievementAridAuthority')}`,
    modalId: 'generic-achievement',
  },
  ACHIEVEMENT_PARTY_UP: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('achievementPartyUp')}`,
    modalId: 'generic-achievement',
  },
  ACHIEVEMENT_PARTY_ON: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('achievementPartyOn')}`,
    modalId: 'generic-achievement',
  },
  ACHIEVEMENT_BEAST_MASTER: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('beastAchievement')}`,
    modalId: 'generic-achievement',
    data: {
      message: $t => $t('achievement'),
      modalText: $t => $t('mountAchievement'),
    },
  },
  ACHIEVEMENT_MOUNT_MASTER: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('mountAchievement')}`,
    modalId: 'generic-achievement',
    data: {
      message: $t => $t('achievement'),
      modalText: $t => $t('mountAchievement'),
    },
  },
  ACHIEVEMENT_TRIAD_BINGO: {
    achievement: true,
    label: $t => `${$t('achievement')}: ${$t('triadBingoAchievement')}`,
    modalId: 'generic-achievement',
    data: {
      message: $t => $t('achievement'),
      modalText: $t => $t('triadBingoAchievement'),
    },
  },
};

export default {
  components: {
    yesterdailyModal,
    wonChallenge,
    ultimateGear,
    streak,
    rebirth,
    joinedGuild,
    joinedChallenge,
    invitedFriend,
    newStuff,
    death,
    lowHealth,
    levelUp,
    chooseClass,
    armoireEmpty,
    questCompleted,
    questInvitation,
    testing,
    testingletiant,
    rebirthEnabled,
    dropsEnabled,
    contributor,
    loginIncentives,
    verifyUsername,
    genericAchievement,
    lostMasterclasser,
    mindOverMatter,
    justAddWater,
  },
  mixins: [notifications, guide],
  data () {
    // Levels that already display modals and should not trigger generic Level Up
    const unlockLevels = {
      3: 'drop system',
      10: 'class system',
      50: 'Orb of Rebirth',
    };

    // Avoid showing the same notiication more than once
    const lastShownNotifications = [];
    const alreadyReadNotification = [];

    // A list of notifications handled by this component,
    // NOTE: Those not listed here won't be handled at all!
    const handledNotifications = {};

    [
      'GUILD_PROMPT', 'DROPS_ENABLED', 'REBIRTH_ENABLED', 'WON_CHALLENGE', 'STREAK_ACHIEVEMENT',
      'ULTIMATE_GEAR_ACHIEVEMENT', 'REBIRTH_ACHIEVEMENT', 'GUILD_JOINED_ACHIEVEMENT',
      'CHALLENGE_JOINED_ACHIEVEMENT', 'INVITED_FRIEND_ACHIEVEMENT', 'NEW_CONTRIBUTOR_LEVEL',
      'CRON', 'SCORED_TASK', 'LOGIN_INCENTIVE', 'ACHIEVEMENT_ALL_YOUR_BASE', 'ACHIEVEMENT_BACK_TO_BASICS',
      'GENERIC_ACHIEVEMENT', 'ACHIEVEMENT_PARTY_UP', 'ACHIEVEMENT_PARTY_ON', 'ACHIEVEMENT_BEAST_MASTER',
      'ACHIEVEMENT_MOUNT_MASTER', 'ACHIEVEMENT_TRIAD_BINGO', 'ACHIEVEMENT_DUST_DEVIL', 'ACHIEVEMENT_ARID_AUTHORITY',
    ].forEach(type => {
      handledNotifications[type] = true;
    });

    return {
      yesterDailies: [],
      levelBeforeYesterdailies: 0,
      notificationData: {},
      unlockLevels,
      lastShownNotifications,
      alreadyReadNotification,
      isRunningYesterdailies: false,
      nextCron: null,
      handledNotifications,
    };
  },
  computed: {
    // https://stackoverflow.com/questions/42133894/vue-js-how-to-properly-watch-for-nested-properties/42134176#42134176
    ...mapState({
      user: 'user.data',
      userHp: 'user.data.stats.hp',
      userGp: 'user.data.stats.gp',
      userMp: 'user.data.stats.mp',
      userNotifications: 'user.data.notifications',
      userAchievements: 'user.data.achievements', // @TODO: does this watch deeply?
      armoireEmpty: 'user.data.flags.armoireEmpty',
      questCompleted: 'user.data.party.quest.completed',
    }),
    userClassSelect () {
      return !this.user.flags.classSelected && this.user.stats.lvl >= 10;
    },
    userHasClass () {
      return this.$store.getters['members:hasClass'](this.user);
    },
    invitedToQuest () {
      return this.user.party.quest.RSVPNeeded && !this.user.party.quest.completed;
    },
    userExpAndLvl () {
      return [this.user.stats.exp, this.user.stats.lvl];
    },
  },
  watch: {
    userHp (after, before) {
      if (this.user.needsCron) return;
      if (after <= 0) {
        this.showDeathModal();
        // @TODO: {keyboard:false, backdrop:'static'}
      } else if (after <= 30 && !this.user.flags.warnedLowHealth) {
        this.$root.$emit('bv::show::modal', 'low-health');
        // @TODO: {keyboard:false, backdrop:'static', controller:'UserCtrl'}
      }
      if (after === before) return;
      if (this.user.stats.lvl === 0) return;
      this.hp(after - before, 'hp');

      if (after < 0) this.playSound('Minus_Habit');
    },
    userGp (after, before) {
      if (after === before) return;
      if (this.user.stats.lvl === 0) return;

      const money = after - before;
      let bonus;
      if (this.user._tmp) {
        bonus = this.user._tmp.streakBonus || 0;
      }
      this.gp(money, bonus || 0);

      //  Append Bonus
      if (money > 0 && Boolean(bonus)) {
        if (bonus < 0.01) bonus = 0.01;
        this.streak(`+ ${this.coins(bonus)}`);
        delete this.user._tmp.streakBonus;
      }
    },
    userMp (after, before) {
      if (after === before) return;
      if (!this.userHasClass) return;

      const mana = after - before;
      this.mp(mana);
    },
    userClassSelect (after) {
      if (this.user.needsCron) return;
      if (!after) return;
      this.$root.$emit('bv::show::modal', 'choose-class');
      // @TODO: {controller:'UserCtrl', keyboard:false, backdrop:'static'}
    },
    userNotifications (after) {
      if (this.user.needsCron) return;
      this.handleUserNotifications(after);
    },
    armoireEmpty (after, before) {
      if (after === before || after === false) return;
      this.$root.$emit('bv::show::modal', 'armoire-empty');
    },
    questCompleted () {
      if (!this.questCompleted) return;
      this.$root.$emit('bv::show::modal', 'quest-completed');
      this.playSound('Achievement_Unlocked');
    },
    invitedToQuest (after) {
      if (after !== true) return;
      this.$root.$emit('bv::show::modal', 'quest-invitation');
    },
    userExpAndLvl (after, before) {
      this.displayUserExpAndLvlNotifications(after[0], before[0], after[1], before[1]);
    },
  },
  mounted () {
    Promise.all([
      this.$store.dispatch('user:fetch'),
      this.$store.dispatch('tasks:fetchUserTasks'),
    ]).then(() => {
      // @TODO: This is a timeout to ensure dom is loaded
      window.setTimeout(() => {
        this.runForcedModals();
      }, 2000);

      this.debounceCheckUserAchievements();

      // Do not remove the event listener as it's live for the entire app lifetime
      document.addEventListener('mousemove', this.checkNextCron);
      document.addEventListener('touchstart', this.checkNextCron);
      document.addEventListener('mousedown', this.checkNextCron);
      document.addEventListener('keydown', this.checkNextCron);
    });
  },
  beforeDestroy () {
    document.removeEventListener('mousemove', this.checkNextCron);
    document.removeEventListener('touchstart', this.checkNextCron);
    document.removeEventListener('mousedown', this.checkNextCron);
    document.removeEventListener('keydown', this.checkNextCron);
  },
  methods: {
    runForcedModals () {
      if (!this.user.flags.verifiedUsername) return this.$root.$emit('bv::show::modal', 'verify-username');

      return this.runYesterDailies();
    },
    showDeathModal () {
      this.playSound('Death');
      this.$root.$emit('bv::show::modal', 'death');
    },
    showNotificationWithModal (notification, forceToModal) {
      const config = NOTIFICATIONS[notification.type];

      if (!config) {
        return;
      }
      if (config.achievement) {
        this.playSound('Achievement_Unlocked');
      } else if (config.sound) {
        this.playSound(config.sound);
      }

      let data = {};
      if (notification.data) {
        data = notification.data;
      }

      if (!data.modalText && config.data.modalText) {
        data.modalText = config.data.modalText(this.$t);
      }
      if (!data.message && config.data.message) {
        data.message = config.data.message(this.$t);
      }

      this.notificationData = data;
      if (forceToModal) {
        this.$root.$emit('bv::show::modal', config.modalId);
      } else {
        this.text(config.label(this.$t), () => {
          this.notificationData = data;
          this.$root.$emit('bv::show::modal', config.modalId);
        }, false);
      }
    },
    debounceCheckUserAchievements: debounce(function debounceCheck () {
      this.checkUserAchievements();
    }, 700),
    displayUserExpAndLvlNotifications (afterExp, beforeExp, afterLvl, beforeLvl) {
      if (afterExp === beforeExp && afterLvl === beforeLvl) return;

      // XP evaluation
      if (afterExp !== beforeExp) {
        if (this.user.stats.lvl === 0) return;

        const lvlUps = afterLvl - beforeLvl;
        let exp = afterExp - beforeExp;

        if (lvlUps > 0) {
          let level = Math.trunc(beforeLvl);
          exp += toNextLevel(level);

          // loop if more than 1 lvl up
          for (let i = 1; i < lvlUps; i += 1) {
            level += 1;
            exp += toNextLevel(level);
          }
        }
        this.exp(exp);
      }

      // Lvl evaluation
      if (afterLvl !== beforeLvl) {
        if (afterLvl <= beforeLvl || this.$store.state.isRunningYesterdailies) return;
        this.showLevelUpNotifications(afterLvl);
      }
    },
    checkUserAchievements () {
      if (this.user.needsCron) return null;

      // List of prompts for user on changes.
      // Sounds like we may need a refactor here, but it is clean for now
      if (!this.user.flags.welcomed) {
        if (this.$store.state.avatarEditorOptions) {
          this.$store.state.avatarEditorOptions.editingUser = false;
        }
        return this.$root.$emit('bv::show::modal', 'avatar-modal');
      }

      if (this.user.flags.newStuff) {
        return this.$root.$emit('bv::show::modal', 'new-stuff');
      }

      if (this.user.stats.hp <= 0) {
        return this.showDeathModal();
      }

      if (this.questCompleted) {
        this.playSound('Achievement_Unlocked');
        return this.$root.$emit('bv::show::modal', 'quest-completed');
      }

      if (this.userClassSelect) {
        this.playSound('Achievement_Unlocked');
        return this.$root.$emit('bv::show::modal', 'choose-class');
      }

      return null;
    },
    showLevelUpNotifications (newlevel) {
      this.lvl();
      this.playSound('Level_Up');
      if (this.user._tmp && this.user._tmp.drop && this.user._tmp.drop.type === 'Quest') return;
      if (this.unlockLevels[`${newlevel}`]) return;
      if (!this.user.preferences.suppressModals.levelUp) this.$root.$emit('bv::show::modal', 'level-up');
    },
    playSound (sound) {
      this.$root.$emit('playSound', sound);
    },
    checkNextCron: throttle(function checkNextCron () {
      if (
        !this.$store.state.isRunningYesterdailies
        && this.nextCron
        && Date.now() > this.nextCron
      ) {
        Promise.all([
          this.$store.dispatch('user:fetch', { forceLoad: true }),
          this.$store.dispatch('tasks:fetchUserTasks', { forceLoad: true }),
        ]).then(() => this.runYesterDailies());
      }
    }, 1000),
    scheduleNextCron () {
      // Reset the yesterDailies array
      this.yesterDailies = [];

      // Open yesterdailies modal the next time cron runs
      const { dayStart } = this.user.preferences;
      let nextCron = moment().hours(dayStart).minutes(0).seconds(0)
        .milliseconds(0);

      const currentHour = moment().format('H');
      if (currentHour >= dayStart) {
        nextCron = nextCron.add(1, 'day');
      }

      // Setup a listener that executes 10 seconds after the next cron time
      this.nextCron = Number(nextCron.format('x'));
      this.$store.state.isRunningYesterdailies = false;
    },
    async runYesterDailies () {
      if (this.$store.state.isRunningYesterdailies) return;
      this.$store.state.isRunningYesterdailies = true;

      if (!this.user.needsCron) {
        this.scheduleNextCron();
        this.handleUserNotifications(this.user.notifications);
        return;
      }

      const { dailys } = this.$store.state.tasks.data;

      const yesterDay = moment().subtract('1', 'day').startOf('day').add({
        hours: this.user.preferences.dayStart,
      });

      dailys.forEach(task => {
        if (task && task.group.approval && task.group.approval.requested) return;
        if (task.completed) return;
        const due = shouldDo(yesterDay, task);
        if (task.yesterDaily && due) this.yesterDailies.push(task);
      });

      if (this.yesterDailies.length === 0) {
        this.runYesterDailiesAction();
        return;
      }

      this.levelBeforeYesterdailies = this.user.stats.lvl;
      this.$root.$emit('bv::show::modal', 'yesterdaily');
    },
    async runYesterDailiesAction () {
      // Run Cron
      await axios.post('/api/v4/cron');

      // Notifications

      // Sync
      await Promise.all([
        this.$store.dispatch('user:fetch', { forceLoad: true }),
        this.$store.dispatch('tasks:fetchUserTasks', { forceLoad: true }),
      ]);

      this.$store.state.isRunningYesterdailies = false;

      if (
        this.levelBeforeYesterdailies > 0
        && this.levelBeforeYesterdailies < this.user.stats.lvl
      ) {
        this.showLevelUpNotifications(this.user.stats.lvl);
      }

      this.scheduleNextCron();
      this.handleUserNotifications(this.user.notifications);
    },
    async handleUserNotifications (after) {
      if (this.$store.state.isRunningYesterdailies) return;

      if (!after || after.length === 0 || !Array.isArray(after)) return;

      const notificationsToRead = [];
      const scoreTaskNotification = [];

      after.forEach(notification => {
        // This notification type isn't implemented here
        if (!this.handledNotifications[notification.type]) return;

        if (this.lastShownNotifications.indexOf(notification.id) !== -1) {
          return;
        }

        this.lastShownNotifications.push(notification.id);
        if (this.lastShownNotifications.length > 10) {
          this.lastShownNotifications.splice(0, 9);
        }

        let markAsRead = true;

        // @TODO: Use factory function instead
        switch (notification.type) { // eslint-disable-line default-case
          case 'GUILD_PROMPT':
            // @TODO: I'm pretty sure we can find better names for these
            if (notification.data.textletiant === -1) {
              this.$root.$emit('bv::show::modal', 'testing');
            } else {
              this.$root.$emit('bv::show::modal', 'testingletiant');
            }
            break;
          case 'DROPS_ENABLED':
            this.$root.$emit('bv::show::modal', 'drops-enabled');
            break;
          case 'REBIRTH_ENABLED':
            this.$root.$emit('bv::show::modal', 'rebirth-enabled');
            break;
          case 'WON_CHALLENGE':
            this.$root.$emit('bv::show::modal', 'won-challenge');
            break;
          case 'STREAK_ACHIEVEMENT':
            this.text(`${this.$t('streaks')}: ${this.user.achievements.streak}`, () => {
              this.$root.$emit('bv::show::modal', 'streak');
            }, this.user.preferences.suppressModals.streak);
            this.playSound('Achievement_Unlocked');
            break;
          case 'REBIRTH_ACHIEVEMENT':
            this.playSound('Achievement_Unlocked');
            this.$root.$emit('bv::show::modal', 'rebirth');
            break;
          case 'ULTIMATE_GEAR_ACHIEVEMENT':
          case 'GUILD_JOINED_ACHIEVEMENT':
          case 'CHALLENGE_JOINED_ACHIEVEMENT':
          case 'INVITED_FRIEND_ACHIEVEMENT':
          case 'NEW_CONTRIBUTOR_LEVEL':
          case 'ACHIEVEMENT_ALL_YOUR_BASE':
          case 'ACHIEVEMENT_BACK_TO_BASICS':
          case 'ACHIEVEMENT_DUST_DEVIL':
          case 'ACHIEVEMENT_ARID_AUTHORITY':
          case 'ACHIEVEMENT_PARTY_UP':
          case 'ACHIEVEMENT_PARTY_ON':
          case 'ACHIEVEMENT_BEAST_MASTER':
          case 'ACHIEVEMENT_MOUNT_MASTER':
          case 'ACHIEVEMENT_TRIAD_BINGO':
          case 'GENERIC_ACHIEVEMENT':
            this.showNotificationWithModal(notification);
            break;
          case 'CRON':
            if (notification.data) {
              if (notification.data.hp) this.hp(notification.data.hp, 'hp');
              if (notification.data.mp && this.userHasClass) this.mp(notification.data.mp);
            }
            break;
          case 'SCORED_TASK':
            // Search if it is a read notification
            for (let i = 0; i < this.alreadyReadNotification.length; i += 1) {
              if (this.alreadyReadNotification[i] === notification.id) {
                markAsRead = false; // Do not let it be read again
                break;
              }
            }

            // Only process the notification if it is an unread notification
            if (markAsRead) {
              scoreTaskNotification.push(notification);

              // Add to array of read notifications
              this.alreadyReadNotification.push(notification.id);
            }
            break;
          case 'LOGIN_INCENTIVE':
            if (this.user.flags.tour.intro === this.TOUR_END && this.user.flags.welcomed) {
              this.notificationData = notification.data;
              this.$root.$emit('bv::show::modal', 'login-incentives');
            }
            break;
        }

        if (markAsRead) notificationsToRead.push(notification.id);
      });

      const userReadNotifsPromise = false;

      if (notificationsToRead.length > 0) {
        await axios.post('/api/v4/notifications/read', {
          notificationIds: notificationsToRead,
        });
      }

      // @TODO this code is never run because userReadNotifsPromise is never true
      if (userReadNotifsPromise) {
        userReadNotifsPromise.then(() => {
          // Only run this code for scoring approved tasks
          if (scoreTaskNotification.length > 0) {
            const approvedTasks = [];
            for (let i = 0; i < scoreTaskNotification.length; i += 1) {
              // Array with all approved tasks
              const scoreData = scoreTaskNotification[i].data;
              let direction = 'up';
              if (scoreData.direction) direction = scoreData.direction;

              approvedTasks.push({
                params: {
                  task: scoreData.scoreTask,
                  direction,
                },
              });

              // Show notification of task approved
              this.markdown(scoreTaskNotification[i].data.message);
            }

            // Score approved tasks
            // TODO: User.bulkScore(approvedTasks);
          }
        });
      }

      this.debounceCheckUserAchievements();
    },
  },
};
</script>
