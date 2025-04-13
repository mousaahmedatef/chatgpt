this is my html and scss
<div
  class="modal fade show d-block"
  id="publishModal"
  tabindex="-1"
  aria-labelledby="publishModalLabel"
  aria-hidden="true"
>
  <div class="modal-dialog modal-xl">
    <div class="modal-content action-modal">
      <div class="">
        <div class="modal-header">
          <button type="button" class="btn-close position-absolute top-0 start-0 m-3" data-bs-dismiss="modal"
          (click)="onClose(false)" aria-label="Close"></button>

          <!-- Header -->
          <div class="mb-3">
              <img src="assets/icons/feedback2.svg">
              <h5 class="fw-bold mb-0 ms-2">تصويت على التوجيه</h5>
          </div>
        </div>
      </div>
      <div class="modal-body">
        <div>
          <h6 class="fw-bold text-end">{{incidentPollmodel.subject}} </h6>
          <p>{{incidentPollmodel.description}}</p>

      </div>
      </div>
      <div class="modal-footer">
        <button 
        type="button" 
        class="bttn btn-wave" 
        data-bs-toggle="modal" 
        data-bs-target="#exampleModal3"
        [(ngModel)]="vote"
        [value]="true"
        >
          اوافق
        </button>

        <button
        data-bs-toggle="modal" 
        data-bs-target="#exampleModal3"
        [(ngModel)]="vote"
        [value]="true"
        type="button"
        class="bttn btn-cancel"
        >
          لا اوافق
        </button>
      </div>
    </div>
  </div>
</div>

<div class="modal fade new-modals" id="exampleModal3" tabindex="-1" aria-labelledby="exampleModalLabel"
aria-hidden="true">
<div class="modal-dialog modal-dialog-centered">
    <div class="modal-content">

        <div class="modal-body position-relative p-4">
            <!-- Close Button -->
            <button type="button" class="btn-close position-absolute top-0 start-0 m-3" data-bs-dismiss="modal"
                aria-label="Close"></button>

            <!-- Content -->
            <div>
                <h5 class="fw-bold text-center">هل أنت متأكد؟</h5>
                <!-- Buttons -->
                <div class="actions-btns no-border">
                    <button type="submit" class="btn btn-primary yes" (click)="fillPollVote()">نعم</button>
                    <button type="button" class="btn btn-outline-secondary no" data-bs-dismiss="modal"
                        aria-label="Close">لا</button>
                </div>
            </div>

        </div>

    </div>
</div>
</div>


/** Start Modals **/
.modal-dialog {
  .modal-content {
    border-radius: 0;

    .modal-header {
      display: block;
      border: none;
      padding-top: 24px;

      h1 {
        font-size: 35px;
        color: #58595B;
        font-family: 'regular';
      }

      p {
        color: #757575;
        font-size: 1.2rem;
        margin: 0;
      }
    }

    .modal-body {
      padding-inline: 30px;

      .form-group {
        margin-bottom: 20px;
      }
    }

    .modal-footer {
      justify-content: center;
      border: none;
      margin-bottom: 20px;

      button {
        min-height: 54px;
        border-radius: 0px;
      }
    }
  }
  
  

  
}

::ng-deep {
  position: relative;
  z-index: 1000;
}

#exampleModal3{
  z-index: 1100 !important;
}

/** End Modals **/  

please update it 
