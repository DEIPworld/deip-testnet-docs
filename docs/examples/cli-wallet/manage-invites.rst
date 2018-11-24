**************************************
Manage invites to research group
**************************************

As a member who has been invited to research group you can accept or reject the invitation.

1. To list your invites, use ``list_my_research_group_invites``
2. To accept invite use ``approve_research_group_invite`` command; to reject invite use ``reject_research_group_invite`` command. Parameters:

* id of invite
* account who is invited
* broadcast::

    approve_research_group_invite 0 alice true

    reject_research_group_invite 0 alice true