<template>
  <div>
    <approval-modal :task="task" />
    <div
      v-if="approvalRequested && userIsManager"
      class="claim-bottom-message d-flex align-items-center justify-content-around"
    >
      <a
        class="approve-color"
        @click="approve()"
      >{{ $t('approveTask') }}</a>
      <a @click="needsWork()">{{ $t('needsWork') }}</a>
    </div>
    <div
      v-else-if="multipleApprovalsRequested && userIsManager"
      class="claim-bottom-message d-flex align-items-center"
    >
      <a @click="showRequests()">{{ $t('viewRequests') }}</a>
    </div>
    <div
      v-else-if="!(userIsAssigned && task.group.approval.approved
        && !task.completed && task.type !== 'habit')"
      class="claim-bottom-message d-flex align-items-center"
    >
      <div
        class="mr-auto ml-2"
        v-html="message"
      ></div>
      <div
        v-if="userIsManager && task.group.assignedUsers.length === 1"
        class="ml-auto mr-2"
      >
        <a
          class="unclaim-color"
          @click="unassign()"
        >{{ $t('unassign') }}</a>
      </div>
      <div
        v-if="!userHasClaimed && !task.completed && task.group.assignedUsers.length === 0"
        class="ml-auto mr-2"
      >
        <a
          class="claim-color"
          @click="claim()"
        >{{ $t('claim') }}</a>
      </div>
      <div
        v-if="userHasClaimed && !approvalRequested && !task.completed"
        class="ml-auto mr-2"
      >
        <a
          class="unclaim-color"
          @click="unclaim()"
        >{{ $t('removeClaim') }}</a>
      </div>
    </div>
    <div
      v-if="userIsAssigned && task.group.approval.approved
        && !task.completed && task.type !== 'habit'"
      class="claim-bottom-message d-flex align-items-center justify-content-around"
    >
      <a
        class="approve-color"
        @click="$emit('claimRewards')"
      >{{ $t('claimRewards') }}</a>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  @import '~@/assets/scss/colors.scss';
  .claim-bottom-message {
    background-color: $gray-700;
    border-bottom-left-radius: 2px;
    border-bottom-right-radius: 2px;
    color: $gray-200;
    font-size: 12px;
    padding-bottom: 0.25rem;
    padding-top: 0.25rem;
    z-index: 9;
  }

  .approve-color {
    color: $green-10 !important;
  }
  .claim-color {
    color: $blue-10 !important;
  }
  .unclaim-color {
    color: $red-50 !important;
  }
</style>

<script>
import findIndex from 'lodash/findIndex';
import { mapState } from '@/libs/store';
import approvalModal from './approvalModal';
import sync from '@/mixins/sync';

export default {
  components: {
    approvalModal,
  },
  mixins: [sync],
  props: ['task', 'group'],
  computed: {
    ...mapState({ user: 'user.data' }),
    userIsAssigned () {
      return this.task.group.assignedUsers
        && this.task.group.assignedUsers.indexOf(this.user._id) !== -1;
    },
    userHasClaimed () {
      return this.task.group.claimedUsers
        && this.task.group.claimedUsers.indexOf(this.user._id) !== -1;
    },
    message () {
      const { assignedUsers, claimable, claimedUsers } = this.task.group;

      if (claimable) {
        const claimedUsersLength = claimedUsers.length;
        const claimedUsersNames = [];
        if (this.group && this.group.members) {
          claimedUsers.forEach(userId => {
            const index = findIndex(this.group.members, member => member._id === userId);
            const claimedMember = this.group.members[index];
            claimedUsersNames.push(`@${claimedMember.auth.local.username}`);
          });
        }

        if (claimedUsersLength === 1 && !this.userHasClaimed) {
          return this.$t('claimedByUser', { userName: claimedUsersNames[0] });
        } if (claimedUsersLength > 1 && !this.userHasClaimed) {
          return this.$t('claimedByMembers', { userCount: claimedUsersLength });
        } if (claimedUsersLength > 1 && this.userHasClaimed) {
          return this.$t('claimedByYouAndMembers', { userCount: claimedUsersLength - 1 });
        } if (this.userHasClaimed) {
          return this.$t('youClaimedTask');
        }
        return this.$t('taskIsUnclaimed');
      }

      const assignedUsersLength = assignedUsers.length;
      const assignedUsersNames = [];
      if (this.group && this.group.members) {
        assignedUsers.forEach(userId => {
          const index = findIndex(this.group.members, member => member._id === userId);
          const assignedMember = this.group.members[index];
          assignedUsersNames.push(`@${assignedMember.auth.local.username}`);
        });
      }

      if (assignedUsersLength === 1 && !this.userIsAssigned) {
        return this.$t('assignedToUser', { userName: assignedUsersNames[0] });
      } if (assignedUsersLength > 1 && !this.userIsAssigned) {
        return this.$t('assignedToMembers', { userCount: assignedUsersLength });
      } if (assignedUsersLength > 1 && this.userIsAssigned) {
        return this.$t('assignedToYouAndMembers', { userCount: assignedUsersLength - 1 });
      } if (this.userIsAssigned) {
        return this.$t('youAreAssigned');
      }
      return this.$t('taskIsUnassigned');
    },
    userIsManager () {
      if (!this.group) return false;
      if (!this.group.leader && !this.group.managers) return false;
      if (this.group.leader.id === this.user._id || this.group.managers[this.user._id]) return true;
      return false;
    },
    approvalRequested () {
      if (
        (this.task.approvals && this.task.approvals.length === 1)
        || (this.task.group && this.task.group.approval && this.task.group.approval.requested)
      ) {
        return true;
      }
      return false;
    },
    multipleApprovalsRequested () {
      if (this.task.approvals && this.task.approvals.length > 1) return true;
      return false;
    },
  },
  methods: {
    async claim () {
      let taskId = this.task._id;
      // If we are on the user task
      if (this.task.userId) {
        taskId = this.task.group.taskId;
      }

      await this.$store.dispatch('tasks:claimTask', { taskId });
      this.task.group.claimedUsers.push(this.user._id);
      this.sync();
    },
    async unclaim () {
      let taskId = this.task._id;
      // If we are on the user task
      if (this.task.userId) {
        taskId = this.task.group.taskId;
      }

      await this.$store.dispatch('tasks:unclaimTask', { taskId });
      this.task.group.claimedUsers.splice(
        this.task.group.claimedUsers.indexOf(this.user._id), 1,
      );
      this.sync();
    },
    async unassign () { // Only available if there is just a single assigned user
      let taskId = this.task._id;
      const userId = this.task.group.assignedUsers[0];
      // If we are on the user task
      if (this.task.userId) {
        taskId = this.task.group.taskId;
      }

      await this.$store.dispatch('tasks:unassignTask', {
        taskId,
        userId,
      });
      const index = this.task.group.assignedUsers.indexOf(userId);
      this.task.group.assignedUsers.splice(index, 1);

      this.sync();
    },
    async approve () {
      const userIdToApprove = this.task.group.assignedUsers[0];
      await this.$store.dispatch('tasks:approve', {
        taskId: this.task._id,
        userId: userIdToApprove,
      });
      this.task.group.assignedUsers.splice(0, 1);
      this.task.approvals.splice(0, 1);

      this.sync();
    },
    needsWork () {
      const userIdNeedsMoreWork = this.task.group.assignedUsers[0];
      this.$store.dispatch('tasks:needsWork', {
        taskId: this.task._id,
        userId: userIdNeedsMoreWork,
      });
      this.task.approvals.splice(0, 1);

      this.sync();
    },
    showRequests () {
      this.$root.$emit('bv::show::modal', 'approval-modal');
    },
  },
};
</script>
