import {Component, inject, OnDestroy, OnInit} from '@angular/core';
import {Incident} from '../../../ViewModel/DepartmentModules/InternalWorkflow/incident';
import {map, of, Subject, Subscription, switchMap, tap} from 'rxjs';
import {AppLocalStorage} from '../../../Shared/SharedServices/app-local-storage.service';
import {LocalStorageKeysEnum} from '../../../Enum/local-storage-keys-enum';
import {ActivatedRoute, NavigationEnd, Router} from '@angular/router';
import {MatTabsModule} from '@angular/material/tabs';
import {SharedModule} from '../../../Shared/shared.module';
import {GlobalFunctions} from '../../../Shared/global-functions';
import {IncidentService} from '../../../Services/DepartmentModules/InternalWorkflow/incident.service';
import {IncidentStatusEnum} from '../../../Enum/incident-status-enum';
import {AuthService} from '../../AuthModule/services/auth.service';
import {MatDialog, MatDialogConfig} from '@angular/material/dialog';
import {IncidentMessageService} from '../../../Services/DepartmentModules/InternalWorkflow/incident-message.service';
import {IncidentMessage} from '../../../ViewModel/DepartmentModules/InternalWorkflow/incident-message';
import { OutsideEntityService } from '../../../Services/DepartmentModules/AdminModule/Lookups/outside-entity.service';
import { OutsideEntity } from '../../../ViewModel/DepartmentModules/AdminModule/Lookups/outside-entity';
import { Helpers } from '../../../Helpers/helpers';
import { UpdateIsReadModel } from '../../../ViewModel/DepartmentModules/InternalWorkflow/SharedComponents/update-isRead';

@Component({
  selector: 'app-incident',
  standalone: true,
  imports: [
    MatTabsModule,
    SharedModule,
  ],
  providers: [IncidentMessageService, OutsideEntityService],
  templateUrl: './incident.component.html',
  styleUrl: './incident.component.scss',
})
export class IncidentComponent implements OnInit, OnDestroy {
  activatedRoute: ActivatedRoute = inject(ActivatedRoute);
  model: Incident = new Incident();
  $destroy: Subject<void> = new Subject<void>();
  currentRoleKey: number = AppLocalStorage.get(LocalStorageKeysEnum.USER)
    ?.roleKey;
  currententityId: number = AppLocalStorage.get(LocalStorageKeysEnum.USER)
    ?.entityId;
  selectedTabIndex = 0;
  indexOfProcedureTab = 2;
  service: IncidentService = inject(IncidentService);
  incidentMessageService: IncidentMessageService = inject(
    IncidentMessageService,
  );
  router: Router = inject(Router);
  selectedChildRoute!: string;
  authService = inject(AuthService);
  matDialog: MatDialog = inject(MatDialog);
  matDialogConfig: MatDialogConfig = new MatDialogConfig();
  readonly activateNationalPlanRoute = 'activate-national-plan';
  readonly manageTasksRoute = 'tasks';
  readonly recommendActivatePlan = 'recommended-deactivate-plan';
  readonly entityInfoRoute = 'entity-info';
  readonly recommenDeactivateNationalPlanRoute = 'recommended-deactivate-plan';
  nextProcedureRoute = '';
  hideNextProcedureRoutes = [
    'entity-info',
    'leader-order',
    'leader-order-desc',
    'leader-info',
    'leader-info-desc',
  ];
  currentUrlSegments: string[] = [];
  private messageSubscription!: Subscription;

  outsideEntityService = inject(OutsideEntityService);
  outsideEntities: OutsideEntity[] = [];
  updateIsReadModel: UpdateIsReadModel = new UpdateIsReadModel();
  ngOnInit(): void {
    this.loadOutsideEntities(); 
    this.selectedChildRoute = this.getLastURLSegment(this.router.url);
    this.currentUrlSegments = this.router.url.split('/');
    this.router.events.subscribe((event) => {
      if (event instanceof NavigationEnd) {
        this.selectedChildRoute = this.getLastURLSegment(event.url);
        this.currentUrlSegments = this.router.url.split('/');
      }
    });
    this.getModelFromResolver().subscribe((model) => {
      this.model = model ?? new Incident();
      this.setNextProcedureRouteValue();
    });
    this.beginChatConnection();

  }

  setNextProcedureRouteValue() {
    if (this.model.isRecommendActivateNationalPlan()) {
      this.nextProcedureRoute = this.activateNationalPlanRoute;
    } else if (this.model.isActivateNationalPlan()) {
      this.nextProcedureRoute = this.manageTasksRoute;
    } else if (this.model.isRecommendCancelNationalPlan()) {
      this.nextProcedureRoute = this.recommendActivatePlan;
    }
  }

  getModelFromResolver() {
    return of(this.service.selectedIncident.value!)
      .pipe(tap((model) => {
          this.updateIsReadForEntityIfNeeded(model);
        }),
        switchMap((model) => {
          return this.updateIsReadForCivilDefenseIfNeeded(model as unknown as Incident);
        }));
  }

  // getCurrentEntityType(model: Incident): EntityTypesEnum | undefined {
  //   const currententityId = this.authService.loggedInUser.value?.entityId;
  //   if (
  //     model.incidentEntities
  //       ?.map((e) => e.fkOutsideEntityId)
  //       ?.includes(currententityId)
  //   )
  //     return EntityTypesEnum.ENTITY;
  //   else if (
  //     model.incidentLeaderships
  //       ?.map((e) => e.fkOutsideEntityId)
  //       ?.includes(currententityId)
  //   )
  //     return EntityTypesEnum.LEADERSHIP;
  //   else return undefined;
  // }

  updateIsReadForCivilDefenseIfNeeded(incident: Incident) {
    if (incident.incidentKey && !incident.isRead && this.authService.isCivilDefenseRepresentative() && incident.incidentEntities?.length == 0) {
      return this.service
        .updateIsReadForLeader(incident.incidentKey)
        .pipe(map(result => {
          incident.isRead = true;
          incident.isReadDate = result.isReadDate;
          incident.lastUpdateUserId = result.lastUpdateUserId;
          incident.lastUpdateDate = result.lastUpdateDate;
          return incident;
        }))
    } else {
      return of(incident);
    }
  }

  updateIsReadForEntityIfNeeded(model: Incident) {
    const entityId = this.authService.loggedInUser.value?.entityId;
    if (entityId) {
      const isEntitiesNotRead = model.incidentEntities
        ? model.incidentEntities!.filter(
        (e) => e.fkOutsideEntityId == entityId && !e.externalIsRead,
      ).length > 0
        : false;
      const isLeadershipsNotRead = model.incidentLeaderships
        ? model.incidentLeaderships!.filter(
        (l) => l.fkOutsideEntityId == entityId && !l.externalIsRead,
      ).length > 0
        : false;
      if (isLeadershipsNotRead && this.authService.isLeadershipPartyRepresentative()){

        this.updateIsReadModel.entityId = entityId;
        this.updateIsReadModel.incidentId = model.id;
        this.updateIsReadModel.externalReadDate = Helpers.getCurrentLocalDateTime();

        this.service.updateIsRead(this.updateIsReadModel).subscribe();

      }
      if(isEntitiesNotRead && this.authService.isExternalPartyRepresentative())
        this.service.updateIsReadForEntity(model.id).subscribe();
    }
  }

  downloadIncidentPdf() {
    this.service
      .downloadIncidentDocument(this.model.incidentKey)
      .subscribe((res) => {
        if (res) GlobalFunctions.viewFile(this.model.incidentKey + '.pdf', res, this.matDialog);
      });
  }

  isActivateNationalPlan() {
    return this.model.fkStatusId == IncidentStatusEnum.ACTIVATE_NATIONAL_PLAN;
  }

  isCloseNationalPlan() {
    return this.model.fkStatusId == IncidentStatusEnum.CLOSE_NATIONAL_PLAN;
  }
  isForInforming() {
    return this.model.fkStatusId == IncidentStatusEnum.FOR_INFORMING;
  }

  showTasksTab() {
    return this.isActivateNationalPlan() || this.isCloseNationalPlan();
  }
  showActivateNationalPlanTab() {
    return !this.isForInforming();
  }

  getLastURLSegment(url: string): string {
    let lastOccurrence = url.lastIndexOf('/');
    let secondToLast = -1;
    if (lastOccurrence > -1) {
      secondToLast = url.lastIndexOf('/', lastOccurrence - 1);
    }
    const segmentsCount = url.match(/\//g)?.length;
    let x = '';
    if (segmentsCount == 4) {
      x = url.substring(lastOccurrence + 1, url.length);
    } else if (segmentsCount == 5) {
      x = url.substring(secondToLast + 1, lastOccurrence);
    }

    return x;
  }

  showNextProcedureButton() {
    return (
      this.selectedChildRoute != this.nextProcedureRoute &&
      !this.isAnyRoutePartOfCurrentUrl() &&
      (this.model.isRecommendActivateNationalPlan() ||
        this.model.isActivateNationalPlan() ||
        (this.model.isRecommendCancelNationalPlan() && this.authService.isIncidentLeader(this.model.fkLeaderOutsideEntityId!))) &&
      !(this.model.isActivateNationalPlan() && this.authService.isCivilDefenseRepresentative())
      && !this.authService.isExternalPartyMember()
    );
  }

  goToNextProcedure() {
    this.router.navigate(['./' + this.nextProcedureRoute], {
      relativeTo: this.activatedRoute,
    });
  }

  isAnyRoutePartOfCurrentUrl() {
    return this.hideNextProcedureRoutes.some((x) =>
      this.currentUrlSegments.includes(x),
    );
  }

  showSendInfoButton() {
    return this.selectedChildRoute != this.entityInfoRoute && this.authService.isExternalPartyRepresentative() && this.model.isActivateNationalPlan();
  }

  showCloseNationalPlanButton() {
    return (
      this.authService.isIncidentLeader(this.model?.fkLeaderOutsideEntityId!) &&
      this.model.isActivateNationalPlan() &&
      !this.router.url.includes('deactivate-plan')
    );
  }

  showRecommendCloseNationalPlanButton() {
    const entitiesIds = this.model.incidentLeaderships?.map(x => x.fkOutsideEntityId!)!;
    return (
      (this.model.isActivateNationalPlan() ||
        this.model.isWaitingEditRecommendation()) 
       && this.authService.isIncidentLeadershipPartyRepresentative(entitiesIds)
    );
  }
  showRecommendationsTab() {
    const entitiesIds = this.model.incidentLeaderships?.map(x => x.fkOutsideEntityId!)!;
    return (
      this.authService.isIncidentLeader(this.model?.fkLeaderOutsideEntityId!) || this.authService.isIncidentLeadershipPartyRepresentative(entitiesIds)
    );
  }
  showPollsTab() {
    const entitiesIds = this.model.incidentLeaderships?.map(x => x.fkOutsideEntityId!)!;
    return (
      this.authService.isIncidentLeader(this.model?.fkLeaderOutsideEntityId!) || this.authService.isIncidentLeadershipPartyRepresentative(entitiesIds)
    );
  }
  showInternalEntityHeader(){
    return (this.model.isRecommendActivateNationalPlan() || this.model.isForInforming());
  }
  beginChatConnection() {
    const profileId = this.authService.loggedInUser.value?.profileId;
    this.incidentMessageService.startConnection().then(() => {
      this.incidentMessageService
        .joinIncidentGroup(this.model.incidentKey)
        .then(() => {
          this.getUnreadMessageCount();
          this.messageSubscription = this.incidentMessageService
            .getReceivedMessage()
            .subscribe((message: IncidentMessage) => {
              if (message.creationUserId != profileId) {
                GlobalFunctions.playMessageAudio();
                if (!this.router.url.includes('additional-data'))
                  this.model.chatNotificationCount!++;
              }
            });
        });
    });
  }

  getUnreadMessageCount() {
    const profileId = this.authService.loggedInUser.value?.profileId;
    this.incidentMessageService
      .getUnreadMessageCount(profileId!, this.model.incidentKey, false)
      .then((count) => (this.model.chatNotificationCount = count))
  }
  loadOutsideEntities() {
    this.outsideEntityService.load().subscribe(list => {
      this.outsideEntities = list;
    })
  }
  getOutsideEntityName(id?: number) {
    return this.outsideEntities.filter((o) => o.id == id)[0]?.name;
  }
  ngOnDestroy(): void {
    this.$destroy.next();
    this.$destroy.complete();
    this.$destroy.unsubscribe();
    this.service.selectedIncident.complete();
    if (this.messageSubscription) {
      this.messageSubscription.unsubscribe();
      this.incidentMessageService.endConnection();
    }
  }
}
