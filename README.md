sortBy(column: 'requestDate' | 'statusName' | 'entityName'): void {
  if (this.sortColumn === column) {
    this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
  } else {
    this.sortColumn = column;
    this.sortDirection = 'asc';
  }

  this.informationList.sort((a, b) => {
    let aVal: any;
    let bVal: any;

    switch (column) {
      case 'requestDate':
        aVal = new Date(a.requestDate);
        bVal = new Date(b.requestDate);
        break;
      case 'statusName':
        aVal = (a.statusName || '').toLowerCase();
        bVal = (b.statusName || '').toLowerCase();
        break;
      case 'entityName':
        aVal = (a.entityName || '').toLowerCase();
        bVal = (b.entityName || '').toLowerCase();

        const specialValues = ['multiple', 'zero'];
        const isASpecial = specialValues.includes(aVal);
        const isBSpecial = specialValues.includes(bVal);

        if (isASpecial && !isBSpecial) return 1;
        if (!isASpecial && isBSpecial) return -1;
        if (isASpecial && isBSpecial) return 0;
        break;
    }

    if (aVal < bVal) return this.sortDirection === 'asc' ? -1 : 1;
    if (aVal > bVal) return this.sortDirection === 'asc' ? 1 : -1;
    return 0;
  });
}



<span *ngIf="sortColumn === 'requestDate'">
          {{ sortDirection === 'asc' ? '↑' : '↓' }}
        </span>

if (this.sortColumn === column && this.sortedList === listName) {
    this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
  } else {
    this.sortColumn = column;
    this.sortDirection = 'asc';
    this.sortedList = listName;
  }
        
