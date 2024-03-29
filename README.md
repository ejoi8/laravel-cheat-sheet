# Laravel Cheat Sheet

Save uploaded file/image

```php
 $file = request()->file('avatar');
// current datetime + hashname
$filename = $user->id."_".$file->hashName();

// save file in year month date - easier for archive in future
$directory = "avatar/{$user->created_at->format('Y/m/d')}/{$user->id}";
$file->storeAs($directory,$filename,'public');
```

Get auth user name/id
```php
auth()->user()->name;
auth()->user()->id;
```

Get auth user role
```php
auth()->user()->hasRole('admin');
```

Redirect to rout with notification
```php
return redirect()->route('route.name')->with('success','notification here');
```    

Get current time
```php
\Carbon\Carbon::now();
```
  
Get ip
```php
\Request::ip();
```

Search function
```php   
public function index(Request $request)
{
    $data = $this->carian($request)
    ->where('office_id',auth()->user()->office_id)
    ->paginate(10);

        return view('users.index',compact('data'))
            ->with('i', ($request->input('page', 1) - 1) * 5);
}

public function carian($request)
{
    $data = User::orderBy('id','DESC');

    if ($request->filled('carian')) {
        $data->where('nokp','like','%'.$request->carian);
    }

    if ($request->filled('office_id')) {
        $data->where('office_id',$request->office_id);
    }

    if ($request->filled('department_id')) {
        $data->where('department_id',$request->department_id);
    }

    return $data;
}
```
        
Eloquent GroupBy Report
```php
use Illuminate\Support\Facades\DB;

public function index()
{
        $bil_cawangan = $this->statistik_cawangan()->get()->pluck('penerangan','user_count');
        return view('home',compact('users','bil_cawangan'));
}

public function statistik_cawangan()
{
        return DB::table('users')
                 ->select(DB::raw('ref_table.penerangan, count(*) as user_count'))
                 ->join('ref_table', 'users.cawangan_id', '=', 'ref_table.id')
                 ->groupBy('ref_table.penerangan')->orderBy('user_count','DESC');
}
```
        
Eloquent find user DoesntHave record in other table and with status count
```php           
$users = User::where('cawangan_id',auth()->user()->cawangan_id)
    ->whereDoesntHave('table_x', function(Builder $query) {
        $query->where('status_id',1)
        ->havingRaw('COUNT(status_id) > 0');
```                        

Abort if user doesnt meet the condition. Ex: block user from access others users information
```php             
public function show(Staff $Staff)
{
        ...
        abort_if(auth()->user()->sekat($kad_owner,$cawangan_id_kad_owner),403);
        ...
}

//in Model
public function sekat($owner,$cawangan_id)
{
        if ($this->hasRole('Admin')) {
             return $this->cawangan_id != $cawangan_id;
        }

        if ($this->hasRole('User')) {
             return $this->id != $owner;
        }
}
```

LARAVEL BLADE SYNTAX
```html
<strong>Name:</strong>
{!! Form::text('field_name', null, array('placeholder' => 'field_name','class' => 'form-control' . ($errors->has('field_name') ? ' is-invalid' : null) )) !!}
@if ($errors->has('field_name'))
<div class="invalid-feedback">
    <strong>{{ $errors->first('field_name') }}</strong>
</div>
@endif

<strong>Country:</strong>
{!! Form::select('country_list', $ref_table->pluck('nama','id'),null,['class' => 'form-control'. ($errors->has('country_list') ? ' is-invalid' : null) ,'placeholder' => 'Choose country...']); !!}
@if ($errors->has('country_list'))
<div class="invalid-feedback">
    <strong>{{ $errors->first('country_list') }}</strong>
</div>
@endif


<strong>Password:</strong>
{!! Form::password('password', array('placeholder' => 'Password','class' => 'form-control' . ($errors->has('password') ? ' is-invalid' : null) )) !!}
@if ($errors->has('password'))
<div class="invalid-feedback">
    <strong>{{ $errors->first('password') }}</strong>
</div>
@endif
<strong>Confirm Password:</strong>
{!! Form::password('password_confirmation', array('placeholder' => 'Confirm Password','class' => 'form-control'. ($errors->has('password_confirmation') ? ' is-invalid' : null) )) !!}
@if ($errors->has('password_confirmation'))
<div class="invalid-feedback">
    <strong>{{ $errors->first('password_confirmation') }}</strong>
</div>
@endif


<strong>Tarikh lahir:</strong>
{!! Form::text('tarikh_lahir', null, array('placeholder' => 'Tarikh lahir','class' => 'form-control datemask'. ($errors->has('roles') ? ' is-invalid' : null) )) !!}
@if ($errors->has('tarikh_lahir'))
<div class="invalid-feedback">
    <strong>{{ $errors->first('tarikh_lahir') }}</strong>
</div>
@endif

<strong>Alamat:</strong>
{!! Form::textarea('alamat', null, array('placeholder' => 'Alamat','class' => 'form-control'. ($errors->has('alamat') ? ' is-invalid' : null) )) !!}
@if ($errors->has('alamat'))
<div class="invalid-feedback">
    <strong>{{ $errors->first('alamat') }}</strong>
</div>
@endif

<select class="form-control" id="filter_status" name="filter_status">
    @foreach($statuses as $status)
        <option value="{{ $status }}">{{ $status }}</option>
    @endforeach
</select>


<script>
    $(document).ready(function() {
        //need to include related script such as jquery,select,inputmask,bootstrapDualListbox

        //select2
        $('.selectComponent').select2();
        //Datemask dd/mm/yyyy
        $('.datemask').inputmask('yyyy-mm-dd', { 'placeholder': 'yyyy-mm-dd' })
        //adminlte 3 default collapse card 
        $('#login-akaun').CardWidget('collapse');
        //adminlte 3 default collapse card 
        $('#maklumat-tambahan').CardWidget('collapse');
        //alternative for select2. More convinient to select record
        $("#user_id").bootstrapDualListbox();

        //show filename after select file
            $('input[type="file"]').change(function(e){
                    var fileName = e.target.files[0].name;
                    alert('The file "' + fileName +  '" has been selected.');
                    var $this = $(this);
                    $this.next().html($this.val().split('\\').pop());
                });

    });
</script>
```

Create Laravel logs Traits (create inside app/IzzulTraits/helper.php)
```php
<?php 

namespace App\IzzulTraits;

trait helper
{
        public function logfile($array_data)
        {

                //DATE
                $TarikhLog = date("d-m-Y [h:i:s A]");
                $day = substr($TarikhLog, 0, 2);
                $month = substr($TarikhLog, 3,2);
                $year = substr($TarikhLog, 6, 4);
                //IP
                $ip = $_SERVER['REMOTE_ADDR'];
                //Text Log File
                $myFile = "$year-$month.log";
                $fh = fopen($myFile, 'a') or die("can't open file");
                // $stringData = " \n";
                $stringData = "$year-$month-$day|$TarikhLog|$ip";
                foreach ($array_data as $data) {
                        $stringData = $stringData."|" .$data; 
                }
                $stringData = $stringData."\n";

                fwrite($fh, $stringData);
                fclose($fh);
        }
}

// add in controller: use App\IzzulTraits\helper; 
// add in the controller class: use helper; 
// dont forget to: composer dump autoload
// call inside controller function: $this->logfile([$request->email,'login']);
```
                
Create table
```
php artisan make:migration create_users_table --create=users 
```
OR
```    
php artisan make:model -m Users <-- auto create migration & model.
```

Add column to existing table
```
php artisan make:migration add_votes_to_users_table --table=users
``` 

Rollback/refresh spesifik table
```
php artisan migrate:refresh --path=/database/migrations/fileName.php
```

Inside the generated file:
```php
public function up()  
{  
    Schema::table('users', function (Blueprint $table) {  
            $table->string('profile')->nullable();  
    });  
}  
public function down()  
{  
    Schema::table('users', function (Blueprint $table) {  
            $table->dropColumn(['profile']);  
    });  
}
```
    
SAMPLE IF SET FOREIGN KEY ON EXISTIN TABLE
```php
/**
 * Run the migrations.
 *
 * @return void
 */
public function up()
{
    Schema::disableForeignKeyConstraints();
    Schema::table('user', function (Blueprint $table) {
        $table->unsignedInteger('user_id')->after('user_id')->default('1');

        $table->foreign('user_id')
                ->references('id')->on('user_category')
                ->onDelete('cascade');
    });
}

/**
 * Reverse the migrations.
 *
 * @return void
 */
public function down()
{
    Schema::table('user', function (Blueprint $table) {
        $table->dropForeign('user_user_id_foreign');
        $table->dropColumn(['user_id']);
    });
}
```
        
SAMPLE DROP COLUMN CONTAIN FOREIGN KEY
```php
/**
 * Run the migrations.
 *
 * @return void
 */
public function up()
{
    Schema::table('kad_ahlis', function (Blueprint $table) {
        $table->dropForeign('kad_ahlis_status_id_foreign'); 
        $table->dropForeign('kad_ahlis_last_updated_by_foreign'); 
        $table->dropColumn(['nota','bukti_bayaran','status_id','last_updated_by']);
    });
}

/**
 * Reverse the migrations.
 *
 * @return void
 */
public function down()
{
    Schema::disableForeignKeyConstraints();
    Schema::table('kad_ahlis', function (Blueprint $table) {
        $table->text('nota');
        $table->text('bukti_bayaran');
        $table->unsignedBigInteger('status_id');
        $table->unsignedBigInteger('last_updated_by');

        $table->foreign('status_id')
                ->references('id')->on('ref_statuses')
                ->onDelete('cascade');

        $table->foreign('last_updated_by')
                ->references('id')->on('users')
                ->onDelete('cascade');
    });
}
```

MAKE NULLABLE
```php
 /**
 * Run the migrations.
 *
 * @return void
 */
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable()->change();
    });
}

/**
 * Reverse the migrations.
 *
 * @return void
 */
public function down()
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->change();
    });
}
```

Clear all cache memory
```
php artisan optimize:clear

php artisan cache:clear  
php artisan config:clear
php artisan config:cache  
php artisan view:clear 
```
    
Download Excel (csv)
```php
    $array_data = array();
    $myFile = public_path('location/folder')."/download".date("Y_m_d_H_i_s").".csv"; //save in folder.
    $fh = fopen($myFile, 'a') or die("can't open file");
    $stringData = "";
    foreach ($array_data as $data_column) {
        $stringData = $stringData.$data_column[0].",".$data_column[1].",".$data_column[2].",".$data_column[3]"\n"; 
    }
    $stringData = $stringData."\nTarikh muat turun: ".date("Y-m-d H:i:s");

    fwrite($fh, $stringData);
    fclose($fh);

    //download EXCEL (CSV)
    if (file_exists($myFile)) {
        header('Content-Description: File Transfer');
        header('Content-Type: application/octet-stream');
        header('Content-Disposition: attachment; filename='.basename($myFile));
        header('Content-Transfer-Encoding: binary');
        header('Expires: 0');
        header('Cache-Control: must-revalidate');
        header('Pragma: public');
        header('Content-Length: ' . filesize($myFile));
        ob_clean();
        flush();
        readfile($myFile);
        unlink($myFile); //delete file
        exit;
    }
```

Ajax request
```php
 //controller
 public function find_cawangan_kecil(Request $request)
{
    if ($request->has('cawangan_id')) {
        $cawangan_kecil = ref_cawangan_kecil::where('cawangan_id',$request->cawangan_id)->get();
        return $cawangan_kecil->pluck('penerangan','id');
    }

}

{{-- html component: change the cawangan list to populate the cawangan kecil  --}}
<strong>Cawangan:</strong>
{!! Form::select('cawangan_id', $ref_cawangan->pluck('nama','id'),null,['class' => 'form-control selectComponent dynamic'. ($errors->has('cawangan_id') ? ' is-invalid' : null) ,'placeholder' => 'Pilih jabatan...', 'id' => 'cawangan']); !!}


<strong>Cawangan kecil:</strong>
<select class="form-control selectComponent {{ ($errors->has('cawangan_kecil_id') ? ' is-invalid' : null) }}" name="cawangan_kecil_id" id="cawangan_kecil" placeholder="SILA PILIH CAWANGAN"><option>--Sila Pilih--</option></select>

//script section
$(".container").on('change','#cawangan',function() {
        if($(this).val() != ''){
        $.ajax({
                type: "GET",
                // dataType: "html",
                url: "{{ route('request.find_cawangan_kecil') }}",
                data: {'cawangan_id': $("#cawangan").val() },
                beforeSend: function(){
                     // $("#loading").show();
                   },
                success: function(response) 
                {    
                    // $("#loading").hide();   
                    console.log(response);
                    $("#cawangan_kecil").empty();
                    $("#cawangan_kecil").append('<option>--Sila Pilih Cawangan Kecil--</option>');
                    if(response)
                    {
                        $.each(response,function(key,value){
                            $('#cawangan_kecil').append($("<option/>", {
                               value: key,
                               text: value
                            }));
                        });
                    }
                }
            });
        }
    });
```

Upload file index
```html
<form method="POST" action="#" enctype="multipart/form-data">
    <input type="file" name="attachment">
    <button type="submit" class="btn btn-primary btn-round btn-lg">Hantar</button>
</form>
```
        
Upload sigle file controller
```php 
$user = User::create($input);
if ($request->hasFile('attachment')) {
    $original_file_name = $request->file('attachment')->getClientOriginalName();
    $file_extension = $request->file('attachment')->clientExtension();
    $file_name = date("Ymd_His").'.'.$file_extension;
    $attachment_name = $request->file('attachment')->storeAs('folder_name',$file_name);
    $user->update(['attachment' => $attachment_name]);
}
```
        
Upload multiple file controller
```php
# declare in model & call static function in controller. Ex: $attachments = $MODEL_NAME::$attachments;
$attachments = [
  'attachment_1',
  'attachment_2',
  'attachment_3',
  'attachment_4',
  'attachment_5',
  'attachment_6',
  'attachment_7',
];

$user = User::create($input);
public function uploadAttachment($request,$attachments,$created_object)
{
  $foldername = date("Ym");
  foreach ($attachments as $attachment) {
    if ($request->hasFile($attachment)) {
        $original_file_name = $request->file($attachment)->getClientOriginalName();
        $file_extension = $request->file($attachment)->clientExtension();
        $fileName = date("Ymd_His").'-'.$attachment.'-'.$created_object->id.'.'.$file_extension;
        $dir_file = $request->file($attachment)->storeAs($foldername,$fileName);

        $created_object->update([$attachment => $dir_file]);
    }
  }

}
```

Upload to temporary folder (laravel daily)
```php  
// create model & table for temporary file
php artisan make:model TemporaryFile -m

// schema only add the following
$table->string('folder');
$table->string('filename');

// logic
if ($request->hasFile('media') ) {
    $file = $request->file('media');
    $filename = $file->getClientOriginalName();
    $folder = uniqid().'-'.now()->timestamp;
    $file->storeAs('media/tmp/'.$folder,$filename) ;

    TemporaryFile::create([
        'folder' => $folder,
        'filename' => $filename,
    ]);

    return $folder ;
}
``` 
        
---  
            
# Infyom Laravel CRUD Generator

Generate field from json file. Save in resources/model_schemes/filename.json
        
        # choose which generator want to skip. https://infyom.com/open-source/laravelgenerator/docs/8.0/generator-options#skip-file-generation
        php artisan infyom:scaffold Semakan --fieldsFile=Semakan.json --skip=views,model,migration,scaffold_controller,scaffold_routes,menu,dump-autoload,api_controller,scaffold_requests,api_routes,tests
        
        {
          "name": "name",
          "dbType": "string,255:nullable",
          "htmlType": "text",
          "validations": "required|min:3",
          "searchable": true,
          "fillable": true,
          "primary": false,
          "inForm": true,
          "inIndex": true,
          "inView": true
        },
        {
          "name": "nokp",
          "dbType": "string,255:nullable",
          "htmlType": "text",
          "validations": "required|min:8|max:12",
          "searchable": true,
          "fillable": true,
          "primary": false,
          "inForm": true,
          "inIndex": true,
          "inView": true
        },
        {
          "name": "email",
          "dbType": "string,255:nullable",
          "htmlType": "text",
          "validations": "required|email",
          "searchable": true,
          "fillable": true,
          "primary": false,
          "inForm": true,
          "inIndex": true,
          "inView": true
        },
        {
          "name": "status_id",
          "dbType": "integer:unsigned:foreign,statuses,id",
          "htmlType": "selectTable:statuses:nama,id",
          "relation": "mt1,statuses,status_code,id"
        },
        {
          "name": "date",
          "dbType": "datetime:nullable",
          "htmlType": "date",
          "validations": "required",
          "searchable": true,
          "fillable": true,
          "primary": false,
          "inForm": true,
          "inIndex": true,
          "inView": true
        },
        {
          "name": "attachment_1",
          "dbType": "string,255:nullable",
          "htmlType": "file",
          "validations": "required|mimes:jpeg,bmp,png,pdf|size:5000",
          "searchable": true,
          "fillable": true,
          "primary": false,
          "inForm": true,
          "inIndex": true,
          "inView": true
        },
        
Sample for form element (checkbox & radio not working)
        
        [
            {
                "name": "id",
                "dbType": "increments",
                "htmlType": null,
                "validations": null,
                "searchable": false,
                "fillable": false,
                "primary": true,
                "inForm": false,
                "inIndex": false,
                "inView": false
            },
            {
                "name": "text",
                "dbType": "string",
                "htmlType": "text",
                "validations": "",
                "searchable": true,
                "fillable": true,
                "primary": false,
                "inForm": true,
                "inIndex": true,
                "inView": true
            },
            {
                "name": "email",
                "dbType": "string",
                "htmlType": "email",
                "validations": null,
                "searchable": true,
                "fillable": true,
                "primary": false,
                "inForm": true,
                "inIndex": true,
                "inView": true
            },
            {
                "name": "number",
                "dbType": "string",
                "htmlType": "number",
                "validations": "",
                "searchable": true,
                "fillable": true,
                "primary": false,
                "inForm": true,
                "inIndex": true,
                "inView": true
            },
            {
                "name": "date",
                "dbType": "timestamp",
                "htmlType": "date",
                "validations": "",
                "searchable": true,
                "fillable": true,
                "primary": false,
                "inForm": true,
                "inIndex": true,
                "inView": true
            },
            {
                "name": "file",
                "dbType": "string",
                "htmlType": "file",
                "validations": "",
                "searchable": true,
                "fillable": true,
                "primary": false,
                "inForm": true,
                "inIndex": true,
                "inView": true
            },
            {
                "name": "password",
                "dbType": "string",
                "htmlType": "password",
                "validations": "",
                "searchable": true,
                "fillable": true,
                "primary": false,
                "inForm": true,
                "inIndex": true,
                "inView": true
            },
            {
                "name": "select",
                "dbType": "string",
                "htmlType": "select,option1:val1,option2:val2",
                "validations": "",
                "searchable": true,
                "fillable": true,
                "primary": false,
                "inForm": true,
                "inIndex": true,
                "inView": true
            },
            {
                "name": "radio",
                "dbType": "string",
                "htmlType": "radio:option1,option2",
                "validations": "",
                "searchable": true,
                "fillable": true,
                "primary": false,
                "inForm": true,
                "inIndex": true,
                "inView": true
            },
            {
                "name": "checkbox",
                "dbType": "string",
                "htmlType": "checkbox:value1",
                "validations": "",
                "searchable": true,
                "fillable": true,
                "primary": false,
                "inForm": true,
                "inIndex": true,
                "inView": true
            },
            {
                "name": "textarea",
                "dbType": "string",
                "htmlType": "textarea",
                "validations": "",
                "searchable": true,
                "fillable": true,
                "primary": false,
                "inForm": true,
                "inIndex": true,
                "inView": true
            },
            {
                "name": "created_at",
                "dbType": "timestamp",
                "htmlType": null,
                "validations": null,
                "searchable": false,
                "fillable": false,
                "primary": false,
                "inForm": false,
                "inIndex": false,
                "inView": true
            },
            {
                "name": "updated_at",
                "dbType": "timestamp",
                "htmlType": null,
                "validations": null,
                "searchable": false,
                "fillable": false,
                "primary": false,
                "inForm": false,
                "inIndex": false,
                "inView": true
            }
        ]

Generate New module from existing table without datatable

        # https://infyom.com/open-source/laravelgenerator/docs/8.0/generator-options
        
        php artisan infyom:scaffold $MODEL_NAME --fromTable --tableName=users  --datatables=false --paginate=10 --skip=model --ignoreFields=password,remember_token
        php artisan infyom:scaffold $MODEL_NAME --datatables=false --paginate=10
        
Generate New module from existing table with datatable

        # https://infyom.com/open-source/laravelgenerator/docs/8.0/generator-options

        php artisan infyom:scaffold $MODEL_NAME --fromTable --tableName=users  --datatables=true --ignoreFields=password,remember_token
        php artisan infyom:scaffold $MODEL_NAME --datatables=true
        
Option during create new table

        # https://labs.infyom.com/laravelgenerator/docs/5.3/fields-input-guide
        
        name string text
        description string:nullable textarea
        attachment1 string:nullable file
        date datetime:nullable file
        
        # select from existing table
        # selectTable:tableName:column1,column2
        # Note:where column1 is Option Label and column2 is Option Value
        
        selectTable:users:name,id
        selectTable:categories:title,id
        
## Date filtering in Datatable index page
___

resources/MODULE_FOLDER_NAME/index.blade.php

        <input type="date" id="min" name="min" class="form-control">
        <input type="date" id="max" name="max" class="form-control">
        <input type="submit" id="generate" class="btn btn-primary">

resources/MODULE_FOLDER_NAME/table.php

        $(document).ready(function() {

            // DataTables initialisation
            var table = window.LaravelDataTables["dataTableBuilder"];

            table.on('preXhr.dt',function(e,settings,data){
                data.min = $('#min').val();
                data.max = $('#max').val();
            }); 

            // Event listener to the two range filtering inputs to redraw on input
            $('#generate').click( function(e) {
                table.ajax.reload();
                e.preventDefault();
            } );

        });
        
app/datatables/ModuleDatatables.php

        public function query(Posts $model)
        {
            $kuiri = $model->newQuery();

            if(!is_null(request()->min)) {
                $kuiri->where('created_date', '>=', request()->min);
            }

            if(!is_null(request()->max)) {
                $kuiri->where('created_date', '<=', request()->max);
            }

                return $kuiri;
        }
___   

## Datatable hide column dynamically (infyom adminLTE template)
___

view/$MODULE_VIEW_FOLDER/index.blade.php


            <select name="column_name" id="column_name" class="form-control selectpicker" multiple>
                <option value="0">action</option>
                <option value="1">id</option>
                <option value="2">qrcode</option>
                <option value="3">name</option>
                <option value="4">nokp</option>
                <option value="5">tel</option>
                <option value="6">address</option>
                <option value="7">photo</option>
                <option value="8">vaccine_photo</option>
                <option value="9">email</option>
                <option value="10">stage_photo_status_id</option>
                <option value="11">studio_photo_status_id</option>
                <option value="12">course_id</option>
                <option value="13">course_code</option>
                <option value="14">cert_level_id</option>
                <option value="15">program_type_id</option>
                <option value="16">status_kehadiran_id</option>
                <option value="17">occupation</option>
                <option value="18">robe_set_id</option>
                <option value="19">robe_size_id</option>
                <option value="20">robe_pickup_status_id</option>
                <option value="21">robe_dropoff_status_id</option>
                <option value="22">ref_no</option>
                <option value="23">vaccine_status_id</option>
                <option value="24">status_kehadiran_event_id</option>
                <option value="25">status_permohonan_id</option>
            </select>

        @push('third_party_stylesheets')
                <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-select@1.13.14/dist/css/bootstrap-select.min.css">
        @endpush


        @push('third_party_scripts')
            <script src="https://cdn.jsdelivr.net/npm/bootstrap-select@1.13.14/dist/js/bootstrap-select.min.js"></script>>
            <script type="text/javascript">
                $(document).ready(function() {

                var table = window.LaravelDataTables["dataTableBuilder"];
                $('#column_name').change(function(){

                    var all_column = ["0","1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25"]

                    var remove_column = $('#column_name').val();

                    var remaining_column = all_column.filter(function(obj) { return remove_column.indexOf(obj) == -1; });

                    table.columns(remove_column).visible(false);

                    table.columns(remaining_column).visible(true);

                });

            });
            </script>
        @endpush

___

# Larvel Collective

Open form

        <!-- If your form is going to accept file uploads, add a files option to your array:  -->
        {!! } Form::open(['route' => ['route.name', $user->id]]) !!}
        {!! } Form::open(['action' => ['Controller@method', $user->id]]) !!}
        {!! } Form::open(['url' => 'foo/bar', 'files' => true]) !!}
        <!-- Close form  -->
        {!! Form::close() !!}
Token

        <!-- Token  -->
        {!! Form::token() !!}
        
Text Field

        <!-- Text Field -->
        <div class="form-group col-sm-6">
            {!! Form::label('text', 'Text:') !!}
            {!! Form::text('text', null, ['class' => 'form-control']) !!}
        </div>
Email

        <!-- Email Field -->
        <div class="form-group col-sm-6">
            {!! Form::label('email', 'Email:') !!}
            {!! Form::email('email', null, ['class' => 'form-control']) !!}
        </div>
Number

        <!-- Number Field -->
        <div class="form-group col-sm-6">
            {!! Form::label('number', 'Number:') !!}
            {!! Form::number('number', null, ['class' => 'form-control']) !!}
        </div>
Date

        <!-- Date Field -->
        <div class="form-group col-sm-6">
            {!! Form::label('date', 'Date:') !!}
            {!! Form::text('date', null, ['class' => 'form-control','id'=>'date']) !!}
        </div>

        @push('page_scripts')
            <script type="text/javascript">
                $('#date').datetimepicker({
                    format: 'YYYY-MM-DD HH:mm:ss',
                    useCurrent: true,
                    sideBySide: true
                })
            </script>
        @endpush
Date now

        <!-- Date NowField -->
        <div class="form-group col-sm-6">
            {!! Form::label('date', 'Date Now carbon:') !!}
            {!! Form::date('date', \Carbon\Carbon::now(),['class' => 'form-control','id'=>'date']) !!}
        </div>
File

        <!-- File Field -->
        <div class="form-group col-sm-6">
            {!! Form::label('file', 'File:') !!}
            <div class="input-group">
                <div class="custom-file">
                    {!! Form::file('file', ['class' => 'custom-file-input']) !!}
                    {!! Form::label('file', 'Choose file', ['class' => 'custom-file-label']) !!}
                </div>
            </div>
        </div>
        <div class="clearfix"></div>
Password

        <!-- Password Field -->
        <div class="form-group col-sm-6">
            {!! Form::label('password', 'Password:') !!}
            {!! Form::password('password', ['class' => 'form-control']) !!}
        </div>
Textarea

        <!-- Textarea Field -->
        <div class="form-group col-sm-12 col-lg-12">
            {!! Form::label('textarea', 'Textarea:') !!}
            {!! Form::textarea('textarea', null, ['class' => 'form-control']) !!}
        </div>
Select with value

        <!-- select Field -->
        {{-- null can be replace with "(isset($model)) ? $model->value : null" --}}
        <div class="form-group col-sm-12 col-lg-12">
            {!! Form::label('select', 'Select:') !!}
            {!! Form::select('size', ['L' => 'Large', 'S' => 'Small'], null, ['class' => 'form-control','placeholder' => 'Choose...']); !!}
        </div>
Select with number range

        <!-- select range Field -->
        <div class="form-group col-sm-12 col-lg-12">
            {!! Form::label('selectrange', 'Select range:') !!}
            {!! Form::selectRange('number', 10, 20,null,['class' => 'form-control','placeholder' => 'Choose...']); !!}
        </div>
Select with month range

        <!-- select month Field -->
        <div class="form-group col-sm-12 col-lg-12">
            {!! Form::label('selectmonth', 'Select month:') !!}
            {!! Form::selectMonth('month',null,['class' => 'form-control','placeholder' => 'Choose month...']); !!}
        </div>
Select with grouping

        <!-- select Group Field -->
        <div class="form-group col-sm-12 col-lg-12">
            {!! Form::label('selectgroup', 'Select Group:') !!}
            {!! Form::select('animal',[
            'Cats' => ['leopard' => 'Leopard'],
            'Dogs' => ['spaniel' => 'Spaniel'],
        ],null,['class' => 'form-control','placeholder' => 'Choose...']); !!}
        </div>
Checkbox

        <!-- Checkbox Field -->
        <div class="form-group col-sm-12 col-lg-12">
            {!! Form::label('checkbox', 'Checkbox:',['class' => '']) !!}
            {!! Form::checkbox('Option 1', 'value1',null,['class' => '']); !!}
            {!! Form::label('checkbox', 'Checkbox:',['class' => '']) !!}
            {!! Form::checkbox('Option 1', 'value2',null,['class' => '']); !!}
        </div>
Radio

        <!-- Checkbox Field -->
        <div class="form-group col-sm-12 col-lg-12">
            {!! Form::label('radio', 'Radio 1:',['class' => '']) !!}
            {!! Form::radio('Option 1', 'value1',null,['class' => '']); !!}
            {!! Form::label('radio', 'Radio 2:',['class' => '']) !!}
            {!! Form::radio('Option 1', 'value2',null,['class' => '']); !!}
        </div>
        
        
___   

## Many to many attach
___
        
        // sample data structure schema
        $user->roles()->attach([
            1 => ['expires' => $expires],
            2 => ['expires' => $expires],
        ]);
        
        
        // original data from request ($request->product)
        [
            {
                "id": 3,
                "name": "PowerPoint ohsem",
                "slug": "power-point-ohsem",
                "desc": "lorem ipsum amet",
                "SKU": "",
                "category": "Power Point",
                "price": "69.000",
                "price_sale": "0.000",
                "status_id": 4,
                "qty": 13
            }
        ]
        
        // from request convert to collection for better data handling
        $product_post = collect(json_decode($request->product));
        
        // mapping data base on sample above
        $qtyAttach = $product_post->keyBy('id')
            ->map(function ($item, $key) {
                return [
                    'qty' =>$item->qty,
                ];
            });
        
        $order->products()->attach($qtyAttach);


# Value to ref table

```php
// https://www.youtube.com/watch?v=iJxawP55s6I

// METHOD ONE - if data contains 1,000,000 , update city will run 1,000,000 times

$cities = [];

foreach (User::all() as $user) {
    if (!array_key_exists($user->FIELD, $cities)) {
        $city = City::create(['name' => $user->city]);
        $cities[$user->city] = $city->id;
    }

    $user->update(['city_id' => $cities[$user->city]]);
}

// METHOD TWO - update will run depends on city instead of total users

$cities = [];

foreach (User::all() as $user) {
    if (!array_key_exists($user->FIELD, $cities)) {
        $city = City::create(['name' => $user->city]);
        $cities[$user->city] = $city->id;
    }
}

foreach ($cities as $cityName => $cityId) {
    User::where('city', $cityName)->update(['city_id' => $cityId]);
}


// METHOD THREE - ideal if faced PHP timeout

User::chunk(1000, function($users) use (&$cities) {
    foreach ($users as $user) {
        if (!array_key_exists($user->FIELD, $cities)) {
            $city = City::create(['name' => $user->city]);
            $cities[$user->city] = $city->id;
        }
    
        $user->update(['city_id' => $cities[$user->city]]);
    }
})

```
