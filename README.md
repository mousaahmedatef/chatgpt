sortColumn: string = '';
sortDirection: 'asc' | 'desc' | '' = '';

sortBy(column: string): void {
  if (this.sortColumn === column) {
    this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
  } else {
    this.sortColumn = column;
    this.sortDirection = 'asc';
  }

  this.informationList.sort((a, b) => {
    let aVal = a[column];
    let bVal = b[column];

    // Special rule for 'entityName'
    if (column === 'entityName') {
      const specialValues = ['multiple', 'zero'];
      const isASpecial = specialValues.includes((aVal || '').toLowerCase());
      const isBSpecial = specialValues.includes((bVal || '').toLowerCase());

      if (isASpecial && !isBSpecial) return 1;
      if (!isASpecial && isBSpecial) return -1;
      if (isASpecial && isBSpecial) return 0;
    }

    // Handle requestDate as Date
    if (column === 'requestDate') {
      aVal = new Date(aVal);
      bVal = new Date(bVal);
    } else {
      // Handle statusName, entityName as lowercase string
      aVal = (aVal || '').toString().toLowerCase();
      bVal = (bVal || '').toString().toLowerCase();
    }

    if (aVal < bVal) return this.sortDirection === 'asc' ? -1 : 1;
    if (aVal > bVal) return this.sortDirection === 'asc' ? 1 : -1;
    return 0;
  });
}


but this lines
let aVal = a[column];
      let bVal = b[column];
      
shows this error
Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'IncidentTaskRequest'.
  No index signature with a parameter of type 'string' was found on type 'IncidentTaskRequest'
