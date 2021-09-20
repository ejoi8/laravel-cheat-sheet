# Laravel Cheat Sheet

Get auth user name/id

        auth()->user()->name;
        auth()->user()->id;

Get auth user role

        auth()->user()->hasRole('admin');

Redirect to rout with notification

        return redirect()->route('route.name')->with('success','notification here');
                        
Get current time

        \Carbon\Carbon::now();
  
Get ip

        \Request::ip();

Search function
   
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
        
Eloquent GroupBy Report

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
        
Eloquent find user DoesntHave record in other table and with status count
                
                $users = User::where('cawangan_id',auth()->user()->cawangan_id)
                    ->whereDoesntHave('table_x', function(Builder $query) {
                        $query->where('status_id',1)
                        ->havingRaw('COUNT(status_id) > 0');
                        

Abort if user doesnt meet the condition. Ex: block user from access others users information
                
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
                
                

LARAVEL BLADE SYNTAX

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

Create Laravel logs Traits (create inside app/IzzulTraits/helper.php)

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
                
Create table

    php artisan make:migration create_users_table --create=users 
    OR
    php artisan make:model -m Users <-- auto create migration & model.
    
Add column to existing table

    php artisan make:migration add_votes_to_users_table --table=users
    

Rollback/refresh spesifik table

    php artisan migrate:refresh --path=/database/migrations/fileName.php
    
Inside the generated file:

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
    
SAMPLE IF SET FOREIGN KEY ON EXISTIN TABLE

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
        
SAMPLE DROP COLUMN CONTAIN FOREIGN KEY

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
        
MAKE NULLABLE

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

Clear all cache memory

    php artisan cache:clear  
    php artisan config:clear
    php artisan config:cache  
    php artisan view:clear 
    
Download Excel (csv)

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

        Ajax request

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

---
Upload file index
        
        <form method="POST" action="#" enctype="multipart/form-data">
            <input type="file" name="attachment">
            <button type="submit" class="btn btn-primary btn-round btn-lg">Hantar</button>
        </form>
        
Upload sigle file controller
        
        $user = User::create($input);
        if ($request->hasFile('attachment')) {
            $file_extension = $request->file('attachment')->getClientOriginalName();
            $file_name = date("Ymd_His").'.'.$file_extension;
            $attachment_name = $request->file('attachment')->storeAs('folder_name',$file_name);
            $user->update(['attachment' => $attachment_name]);
        }
        
Upload multiple file controller
        
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
        public function uploadAttachment($request,$attachments,$bantuanKematian)
        {
          foreach ($attachments as $attachment) {
            if ($request->hasFile($attachment)) {
                $file_extension = $request->file($attachment)->getClientOriginalName();
                $file_name = date("Ymd_His").'-'.$attachment.'-'.$created_object->id.'.'.$file_extension;
                $dir_file = $request->file($attachment)->storeAs('foldername',$file_name);
                $created_object->update([$attachment => $dir_file]);
            }
          }

        }
        
---  
            
# Infyom Laravel CRUD Generator

Generate field from json file. Save in resources/model_schemes/filename.json
        
        php artisan infyom:scaffold $MODEL_NAME --fieldsFile=filename.json
        
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


