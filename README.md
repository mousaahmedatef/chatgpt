 <h2 class="accordion-header">
    <button
      [attr.aria-controls]="'s' + entity().id"
      [attr.data-bs-target]="'#s' + entity().id"
      aria-expanded="false"
      class="accordion-button collapsed"
      data-bs-toggle="collapse"
      type="button"
    >
      {{ entity().outsideEntityName }}
      @if(guidanceEntityOfCurrentEntity.isReadDate){
        <span> (تم الاستلام)</span>
      }
      <div class="plus-icon">
        @if(showChekMark()){
          <img alt="" class="send-message ms-2" src="/assets/imgs/checkmarkpure.png" srcset=""/>
        }
        @if(showPendingMark()){
          <img
          class="send-message ms-2"
          src="/assets/actions/History-Rounded1.svg"
          alt=""
          srcset=""
        />
        }
        <img alt="" src="/assets/actions/plus.png" srcset=""/>
      </div>
      <div class="minus-icon">
        @if(showChekMark()){
          <img alt="" class="send-message ms-2" src="/assets/imgs/checkmarkpure.png" srcset=""/>
        }
        @if(showPendingMark()){
          <img
          class="send-message ms-2"
          src="/assets/actions/History-Rounded1.svg"
          alt=""
          srcset=""
        />
        }
        <img alt="" src="/assets/actions/minus.png" srcset=""/>
      </div>
    </button>
  </h2>
  i have this h2 
  when i add this span to it becames like  {{ entity().outsideEntityName }} in first and span in middle and plus or minus in last , so how to make span beside {{ entity().outsideEntityName }}  
