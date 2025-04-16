<button class="reload-btn position-relative bg-transparent rounded-0 border-0 p-0">
            @if(numberOfPolls){
              <span class="position-absolute top-0 start-70 translate-middle badge rounded-pill bg-danger text-white " style="z-index: 10;">{{numberOfPolls}}</span>
            }
            <a routerLinkActive="active-tab" routerLink="polls"><img height="33" width="33" alt="timeline"
              src="/assets/icons/poll.png" /></a>
            
          </button>
