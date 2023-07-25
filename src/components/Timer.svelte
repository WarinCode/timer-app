<script lang="ts">
  import Swal , { type SweetAlertResult } from "sweetalert2";

  type TupleN = [number , number ,number];
  let [hour, minute, second]: TupleN = [0, 0, 0];
  let interval: null | number = null;
  let oneClick:TupleN[0] = 0;
  var n:TupleN[1] = 0;
  let list: HTMLUListElement;
  const note: string[] = [];

  const handleStart = async ():Promise<void> => {
    oneClick += 1;
    if (oneClick === 1) {
      await Swal.fire({
        icon: "success",
        position: "top-end",
        title: "เริ่มจับเวลา!",
        showConfirmButton: false,
        timer: 1200,
      });
      interval = setInterval((): void => {
        second += 1;
        if (second === 60) {
          second = 0;
          minute += 1;
        }
        if (minute === 60) {
          minute = 0;
          hour += 1;
        }
      }, 1000);
    } else if (oneClick > 1) {
      if (oneClick === 3) {
        alert("โปรแกรมหยุดทำงานโปรดกดปุ่มเพื่อเริ่มจับเวลาใหม่!");
        oneClick = 0;
        clearInterval(interval);
      } else {
        await Swal.fire({
          icon: "error",
          title: "เกิดข้อผิดพลาดขึ้น!",
          html: "<p>ไม่สามารถกดปุ่มเริ่มได้เพราะกำลังจับเวลาอยู่ในตอนนี้คุณต้องกดปุ่ม <strong style='color:red; font-weight:bolder;'>\"หยุด\"</strong> เพิ่อหยุดจับเวลา</p>",
        });
      }
    }
  };

  const handlePause = async (): Promise<void> => {
    if (oneClick === 1) {
      await Swal.fire({
        icon: "success",
        position: "top-end",
        title: "กำลังหยุดเวลา!",
        showConfirmButton: false,
        timer: 1200,
      });
      oneClick = 0;
      clearInterval(interval);
    } else {
      await Swal.fire({
        icon: "error",
        position: "center",
        title: "เกิดข้อผิดพลาดขึ้น",
        html: "คุณต้องการเริ่มโปรแกรมจับเวลาใหม่ไหม?",
        showConfirmButton:true,
        showDenyButton:true,
        confirmButtonText:'ต้องการ',
        denyButtonText:'ไม่ต้องการ',
      }).then(async (res:SweetAlertResult<any>):Promise<any> => {
        if(res.isConfirmed){
          await Swal.fire({
              title:'กำลังเริ่มโปรแกรมใหม่',
              timer: 1800,
              timerProgressBar: true,
              showConfirmButton:false,  
              didOpen:() => Swal.showLoading(),
            })
        }
        return res;
      }).then(async (res:SweetAlertResult<any>):Promise<void> => {
        if(res.isConfirmed){
          await Swal.fire({
              icon: 'success',
              text: 'เริ่มต้นโปรแกรมใหม่เรียบร้อย',
              showConfirmButton:false,
            })
          window.location.reload();
        }
      })
    }
  };

  const handleSave = () => {
    interface Time {
      h: number;
      m: number;
      s: number;
    }
    const time: Time = {
      h: hour,
      m: minute,
      s: second,
    };
    Object.freeze(time);
    const { h, m, s }: Time = time;
    const limit:number = 50;

    if (note.length === 0 && oneClick === 0) {
      Swal.fire({
        icon: "error",
        position: "center",
        title: "เกิดข้อผิดพลาดขึ้น",
        html: "ยังไม่มีการเริ่มจับเวลาโปรดกดปุ่ม <strong style='color:blue;'>\"เริ่ม\"</strong> เพิ่อทำการบันทึกเวลา!",
        showConfirmButton: true,
      });
    } else if(note.length >= limit) {
        Swal.fire({
          icon: "error",
          title: "เกิดข้อผิดพลาดขึ้น!",
          html: `<p>รายการบันทึกเต็มไม่สามารถทำการจดบันทึกเวลาได้มากกว่า ${limit} รายการ\nคุณต้องการจะลบรายการบันทึกทั้งหมดหรือไม่เพิ่อเริ่มการบันทึกใหม่?</p>`,
          confirmButtonText:'ลบรายการ',
          denyButtonText:'ไม่ลบรายการ',  
          showDenyButton: true,
          showCancelButton: false,
        }).then(async (res:SweetAlertResult<any>):Promise<any> => {
          if (res.isConfirmed) {
             await Swal.fire({
              title:'กำลังลบรายการทั้งหมด',
              timer: 2600,
              timerProgressBar: true,
              didOpen:() => Swal.showLoading(),
            })
          }
          return res;
        }).then(async (res:SweetAlertResult<any>):Promise<any> => {
          if (res.isConfirmed) {
            note.length = 0;
            n = 0;
            oneClick = 0;
            clearInterval(interval);
            document!.querySelectorAll('#item')!.forEach((divEL:HTMLDivElement):void => {
              divEL!.remove();
            })            
          return res;
        }
      }).then(async (res:SweetAlertResult<any>):Promise<void> => {
        if(res.isConfirmed){
          await Swal.fire({
            position: 'center',
            icon: 'success',
            title: 'ลบรายการเสร็จเรียบร้อย!',
            showConfirmButton: false,
            timer: 2000
          })          
        }
      })
    } else {
      n += 1;
      note.push(
        `${h === 0 ? `` : `${h} ชั่วโมง`} ${
          m === 0 ? `` : `${m} นาที`
        } ${s === 0 ? `` : `${s} วินาที`}
        `
      );
      const li: HTMLLIElement = document!.createElement("li");
      li!.innerHTML = `<div id="item" class="flex flex-col bg-slate-100 w-96 h-20 my-8 align-items-center justify-center rounded-lg border-2 border-zinc-50 shadow-lg"><li class="text-center text-lg text-black"><i class="ri-time-fill"></i> รอบที่ ${n}.) ${note[n - 1]}</li></div>`;
      list!.appendChild(li);
    }
  };
</script>

<div
  class="flex flex-col justify-center bg-slate-200 w-4/12 h-48 text-center p-6 rounded-xl shadow-2xl
  shadow-gray-400 border-2 border-gray-400 border-solid cursor-default"
>
  <h1 class="text-6xl text-gray-900">
    {hour < 10 ? `0${hour}` : hour} : {minute < 10 ? `0${minute}` : minute} : {second <
    10
      ? `0${second}`
      : second}
  </h1>
</div>
<div class="my-12">
  <button
    class="w-36 ms-3 rounded-md bg-red-600/100 p-3 text-lg font-bold text-white shadow-lg hover:bg-red-800"
    on:click={handlePause}><i class="ri-pause-circle-line"></i> หยุด</button
  >
  <button
    class="w-36 rounded-md bg-orange-500 p-3 text-lg font-bold text-white shadow-lg hover:bg-orange-700 ms-3"
    on:click={handleSave}><i class="ri-save-3-line"></i> บันทึก</button
  >
  <button
    class="w-36 ms-3 rounded-md bg-sky-500/100 p-3 text-lg font-bold text-white shadow-lg hover:bg-sky-600"
    on:click={handleStart}><i class="ri-timer-line"></i> เริ่มจับเวลา</button
  >
</div>
<ul bind:this={list} class="cursor-default w-auto">
  <h3 class="text-3xl text-center text-orange-400 font-bold underline mt-1 mb-10">รายการบันทึกเวลา</h3>
</ul>
