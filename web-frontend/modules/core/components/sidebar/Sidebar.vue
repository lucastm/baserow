<template>
  <div class="sidebar">
    <div class="sidebar__inner">
      <a
        ref="userContextAnchor"
        class="sidebar__user"
        @click="
          $refs.userContext.toggle(
            $refs.userContextAnchor,
            'bottom',
            'left',
            isCollapsed ? -4 : -10,
            isCollapsed ? 8 : 16
          )
        "
      >
        <div class="sidebar__user-initials">
          {{ name | nameAbbreviation }}
        </div>
        <div class="sidebar__user-info">
          <div class="sidebar__user-info-top">
            <div class="sidebar__user-name">{{ name }}</div>
            <div class="sidebar__user-icon">
              <i class="fas fa-caret-down"></i>
            </div>
          </div>
          <div class="sidebar__user-email">{{ email }}</div>
        </div>
      </a>
      <Context ref="userContext">
        <div class="context__menu-title">{{ name }}</div>
        <ul class="context__menu">
          <li>
            <a
              @click="
                ;[
                  $refs.settingsModal.show('password'),
                  $refs.userContext.hide(),
                ]
              "
            >
              <i class="context__menu-icon fas fa-fw fa-cogs"></i>
              {{ $t('sidebar.settings') }}
            </a>
            <SettingsModal ref="settingsModal"></SettingsModal>
          </li>
          <li>
            <a @click="logoff()">
              <i class="context__menu-icon fas fa-fw fa-sign-out-alt"></i>
              {{ $t('sidebar.logoff') }}
            </a>
          </li>
        </ul>
      </Context>
      <div class="sidebar__nav">
        <ul class="tree">
          <li
            class="tree__item"
            :class="{
              active: $route.matched.some(({ name }) => name === 'dashboard'),
            }"
          >
            <div class="tree__action sidebar__action">
              <nuxt-link :to="{ name: 'dashboard' }" class="tree__link">
                <div>
                  <i class="tree__icon fas fa-tachometer-alt"></i>
                  <span class="sidebar__item-name">{{
                    $t('sidebar.dashboard')
                  }}</span>
                </div>
              </nuxt-link>
            </div>
          </li>
          <li class="tree__item">
            <div class="tree__action sidebar__action">
              <a class="tree__link" @click="$refs.trashModal.show()">
                <div>
                  <i class="tree__icon fas fa-trash"></i>
                  <span class="sidebar__item-name">{{
                    $t('sidebar.trash')
                  }}</span>
                </div>
              </a>
              <TrashModal ref="trashModal"></TrashModal>
            </div>
          </li>
          <li v-if="isStaff" class="tree__item">
            <div
              class="tree__action sidebar__action"
              :class="{ 'tree__action--disabled': isAdminPage }"
            >
              <a class="tree__link" @click.prevent="admin()">
                <i class="tree__icon fas fa-users-cog"></i>
                <span class="sidebar__item-name">{{
                  $t('sidebar.admin')
                }}</span>
              </a>
            </div>
            <ul v-show="isAdminPage" class="tree sidebar__tree">
              <li
                v-for="adminType in sortedAdminTypes"
                :key="adminType.type"
                class="tree__item"
                :class="{
                  active: $route.matched.some(
                    ({ name }) => name === adminType.routeName
                  ),
                }"
              >
                <div class="tree__action sidebar__action">
                  <nuxt-link
                    :to="{ name: adminType.routeName }"
                    class="tree__link"
                  >
                    <i
                      class="tree__icon fas"
                      :class="'fa-' + adminType.iconClass"
                    ></i>
                    <span class="sidebar__item-name">{{
                      adminType.getName()
                    }}</span>
                  </nuxt-link>
                </div>
              </li>
            </ul>
          </li>
          <template v-if="hasSelectedGroup && !isCollapsed">
            <li class="tree__item margin-top-2">
              <div class="tree__action">
                <a
                  ref="groupSelectToggle"
                  class="tree__link tree__link--group"
                  @click="
                    $refs.groupSelect.toggle(
                      $refs.groupSelectToggle,
                      'bottom',
                      'left',
                      0
                    )
                  "
                  >{{ selectedGroup.name }}</a
                >
                <GroupsContext ref="groupSelect"></GroupsContext>
              </div>
            </li>
            <li v-if="selectedGroup.permissions === 'ADMIN'" class="tree__item">
              <div class="tree__action">
                <a class="tree__link" @click="$refs.groupMembersModal.show()">
                  <i class="tree__icon tree__icon--type fas fa-users"></i>

                  {{ $t('sidebar.inviteOthers') }}
                </a>
                <GroupMembersModal
                  ref="groupMembersModal"
                  :group="selectedGroup"
                ></GroupMembersModal>
              </div>
            </li>
            <ul class="tree">
              <component
                :is="getApplicationComponent(application)"
                v-for="application in applications"
                :key="application.id"
                v-sortable="{
                  id: application.id,
                  update: orderApplications,
                  handle: '[data-sortable-handle]',
                  marginTop: -1.5,
                }"
                :application="application"
                :group="selectedGroup"
              ></component>
            </ul>
            <li class="sidebar__new-wrapper">
              <a
                ref="createApplicationContextLink"
                class="sidebar__new"
                @click="
                  $refs.createApplicationContext.toggle(
                    $refs.createApplicationContextLink
                  )
                "
              >
                <i class="fas fa-plus"></i>
                {{ $t('action.createNew') }}
              </a>
            </li>
            <CreateApplicationContext
              ref="createApplicationContext"
              :group="selectedGroup"
            ></CreateApplicationContext>
          </template>
          <template v-else-if="!hasSelectedGroup && !isCollapsed">
            <li v-if="groups.length === 0" class="tree_item margin-top-2">
              <p>{{ $t('sidebar.errorNoGroup') }}</p>
            </li>
            <li
              v-for="(group, index) in groups"
              :key="group.id"
              class="tree__item"
              :class="{ 'margin-top-2': index === 0 }"
            >
              <div class="tree__action tree__action--has-right-icon">
                <a
                  class="tree__link tree__link--group"
                  @click="$store.dispatch('group/select', group)"
                  >{{ group.name }}</a
                >
                <i class="tree__right-icon fas fa-arrow-right"></i>
              </div>
            </li>
            <li class="sidebar__new-wrapper">
              <a class="sidebar__new" @click="$refs.createGroupModal.show()">
                <i class="fas fa-plus"></i>
                {{ $t('sidebar.createGroup') }}
              </a>
            </li>
            <CreateGroupModal ref="createGroupModal"></CreateGroupModal>
          </template>
        </ul>
      </div>
      <div class="sidebar__foot">
        <div class="sidebar__logo">
          <img
            height="14"
            src="@baserow/modules/core/static/img/logo.svg"
            alt="Baserow logo"
          />
        </div>
        <a
          class="sidebar__collapse-link"
          @click="$store.dispatch('sidebar/toggleCollapsed')"
        >
          <i
            class="fas"
            :class="{
              'fa-angle-double-right': isCollapsed,
              'fa-angle-double-left': !isCollapsed,
            }"
          ></i>
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapState } from 'vuex'

import { notifyIf } from '@baserow/modules/core/utils/error'
import SettingsModal from '@baserow/modules/core/components/settings/SettingsModal'
import SidebarApplication from '@baserow/modules/core/components/sidebar/SidebarApplication'
import CreateApplicationContext from '@baserow/modules/core/components/application/CreateApplicationContext'
import GroupsContext from '@baserow/modules/core/components/group/GroupsContext'
import CreateGroupModal from '@baserow/modules/core/components/group/CreateGroupModal'
import GroupMembersModal from '@baserow/modules/core/components/group/GroupMembersModal'
import TrashModal from '@baserow/modules/core/components/trash/TrashModal'

export default {
  name: 'Sidebar',
  components: {
    SettingsModal,
    CreateApplicationContext,
    SidebarApplication,
    GroupsContext,
    CreateGroupModal,
    GroupMembersModal,
    TrashModal,
  },
  computed: {
    /**
     * Because all the applications that belong to the user are in the store we will
     * filter on the selected group here.
     */
    applications() {
      return this.$store.getters['application/getAllOfGroup'](
        this.selectedGroup
      ).sort((a, b) => a.order - b.order)
    },
    adminTypes() {
      return this.$registry.getAll('admin')
    },
    sortedAdminTypes() {
      return Object.values(this.adminTypes)
        .slice()
        .sort((a, b) => a.getOrder() - b.getOrder())
    },
    /**
     * Indicates whether the current user is visiting an admin page.
     */
    isAdminPage() {
      return Object.values(this.adminTypes).some((adminType) => {
        return this.$route.matched.some(
          ({ name }) => name === adminType.routeName
        )
      })
    },
    ...mapState({
      allApplications: (state) => state.application.items,
      groups: (state) => state.group.items,
      selectedGroup: (state) => state.group.selected,
    }),
    ...mapGetters({
      isStaff: 'auth/isStaff',
      name: 'auth/getName',
      email: 'auth/getUsername',
      hasSelectedGroup: 'group/hasSelected',
      isCollapsed: 'sidebar/isCollapsed',
    }),
  },
  methods: {
    getApplicationComponent(application) {
      return this.$registry
        .get('application', application.type)
        .getSidebarComponent()
    },
    logoff() {
      this.$store.dispatch('auth/logoff')
      this.$nuxt.$router.push({ name: 'login' })
    },
    /**
     * Called when the user clicks on the admin menu. Because there isn't an
     * admin page it will navigate to the route of the first registered admin
     * type.
     */
    admin() {
      // If the user is already on an admin page we don't have to do anything because
      // the link is disabled.
      if (this.isAdminPage) {
        return
      }

      if (this.sortedAdminTypes.length > 0) {
        this.$nuxt.$router.push({ name: this.sortedAdminTypes[0].routeName })
      }
    },
    async orderApplications(order, oldOrder) {
      try {
        await this.$store.dispatch('application/order', {
          group: this.selectedGroup,
          order,
          oldOrder,
        })
      } catch (error) {
        notifyIf(error, 'application')
      }
    },
  },
}
</script>

<i18n>
{
  "en":{
    "sidebar":{
      "createGroup": "Create group",
      "inviteOthers": "Invite others",
      "logoff": "Logoff",
      "errorNoGroup": "You don’t have any groups.",
      "admin": "Admin",
      "dashboard": "Dashboard",
      "trash": "Trash",
      "settings": "Settings"
    }
  },
  "fr":{
    "sidebar":{
      "createGroup": "Créer un groupe",
      "inviteOthers": "Envoyer une invitation",
      "logoff": "Se déconnecter",
      "errorNoGroup": "Vous n'avez aucun groupe.",
      "admin": "Administration",
      "dashboard": "Accueil",
      "trash": "Corbeille",
      "settings": "Profil"
    }
  }
}
</i18n>
