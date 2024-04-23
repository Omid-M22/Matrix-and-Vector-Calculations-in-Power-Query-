# Matrix and Vector Calculations in Power Query 

Power Query supports a wide range of data types and calculations, but matrix and vector-related calculations such as multiplication, inversion, and determinant computation can be challenging. These tasks are rarely required in daily tasks, but for those who need them, I have developed some new functions. I will introduce these functions here.

Note: In all the codes below, matrices are represented as tables, and vectors are represented as lists.

VectorAddConstant: Adds or subtracts a specific value from all elements in a vector.
```powerquery-m
(Vector as list ,Constant as number ) as list=> List.Transform(Vector, each _+Constant)
```

VectorMultiplyConstant: Multiplies or divides all elements in a vector by a specific value.
```powerquery-m
(Vector as list ,Constant as number ) as list=> List.Transform(Vector, each _*Constant)
```
SumProduct: Similar to Excel's SUMPRODUCT function.
```powerquery-m
(Vector1 as list, Vector2 as list) as number =>
  List.Sum(List.Transform(List.Positions(Vector1), each Vector1{_} * Vector2{_}))
```

MatrixZeros: This creates an n-dimensional matrix where all values are zero.
```powerquery-m
(Constant as number ) as table => Table.FromColumns(List.Repeat({List.Repeat({0},Constant)},Constant))
```

MatrixEye: Creates an n-dimensional diagonal matrix.
Version 1:
```powerquery-m
(Constant as number) as table =>
  Table.FromColumns(
    List.Split(
      List.Transform(
        {1 .. Constant * Constant}, 
        each if Number.Mod(_, Constant + 1) = 1 then 1 else 0
      ), 
      Constant
    )
  )
```

Version 2:
```powerquery-m
(Constant as number) as table =>
  Table.FromColumns(
    List.Transform(
      {1 .. Constant}, 
      each List.InsertRange(List.Repeat({0}, Constant - 1), _ - 1, {1})
    )
  )
```
MutrixMultiply: Like MMult in Excel for multiplying two matrix
```powerquery-m
(MatrixA as table, MatrixB as table) as table =>
  let
    SumProduct = (Vector1, Vector2) =>
      List.Sum(List.Transform(List.Positions(Vector1), each Vector1{_} * Vector2{_})), 
    A = Table.ToRows(MatrixA), 
    B = Table.ToColumns(MatrixB), 
    MMULT = List.Generate(
      () => [i = 0, j = 0], 
      each [j] < List.Count(B), 
      each if [i] = List.Count(A) - 1 then [i = 0, j = [j] + 1] else [i = [i] + 1, j = [j]], 
      each SumProduct(A{[i]}, B{[j]})
    ), 
    ConvToTable = Table.FromColumns(List.Split(MMULT, 3))
  in
    ConvToTable
```


MatrixDeterminant: Calculates the determinant of a matrix.
```powerquery-m
(Matrix) =>
  let
    Dim = Table.RowCount(Matrix), 
    Data = Table.ToRows(Matrix), 
    SignData = List.Transform(
      {1 .. Dim}, 
      each (if Number.IsEven(_ - 1) then 1 else - 1) * Data{0}{_ - 1}
    ), 
    Det = 
      if Dim = 2 then
        Data{0}{0} * Data{1}{1} - Data{0}{1} * Data{1}{0}
      else
        List.Sum(
          List.Transform(
            {0 .. Dim - 1}, 
            each SignData{_}
              * @MatrixِDeterminal(
                Table.RemoveColumns(Table.RemoveRows(Matrix, 0), Table.ColumnNames(Matrix){_})
              )
          )
        )
  in
    Det
```

