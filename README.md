# postgres

SELECT users.id
FROM users
INNER JOIN messages 
ON messages.send_by = users.id OR messages.receive_by = users.id
GROUP BY users.id
ORDER BY users.id

SELECT users.id, users.last_name, users.first_name, MAX(messages.send_at) AS OK
FROM "users"  
INNER JOIN messages 
ON messages.send_by = users.id OR messages.receive_by = users.id
WHERE messages.send_by = 59 or messages.receive_by = 59
GROUP BY users.id
ORDER BY OK DESC





SELECT users.id, users.last_name, users.first_name, messages.send_at 
FROM users
INNER JOIN messages
ON messages.send_by = users.id
OR messages.receive_by = users.id
GROUP BY users.id, messages.send_at
ORDER BY messages.send_at DESC



SELECT DISTINCT users.id, users.* 
FROM "users" 
INNER JOIN messages 
ON (messages.send_by = users.id 
OR messages.receive_by = users.id) 
ORDER BY messages.send_at DESC

    # GET users/with_messagess
def with_messages
  # byebug
  @message = User.joins("INNER JOIN messages ON messages.send_by = users.id OR messages.receive_by = users.id").where("messages.send_by = ? OR messages.receive_by = ?", current_user.id, current_user.id).order('recent_day DESC').select('users.id, MAX (messages.send_at) AS recent_day, users.last_name, users.first_name, nickname, email, birthdate, phone, gender, alternate_phone, emergency_contact_name, emergency_contact_phone, usapa_member_num, canada_member_num, is_instructor, iptpa_certified_instructor, usapa_certified_ref, usapa_ambassador, play_singles, play_doubles, play_mixed, shirt_size, stripe_account_status, stripe_account_type, stripe_user_id, publishable_key, secret_key').group('users.id')
  json_response(@message)
end
---------------------------------------------------------------------
APUNTES MessagesController < ApiController
module Api::V1
  class MessagesController < ApiController
    before_action :authenticate_user!

    require "json"

    def index
      #code
    end

    # GET /v1/messages/{id}
    def show
      @message = current_user.messages_received.find_by(id: message_params[:id])
      @message.present? ? json_response(@message) : (render :status => 404, :json => {:message => "Message doesn't exist"}.to_json)
    end

    # POST /v1/messages/{content, send_by, send_at, receive_by}
    def create
      if current_user.id.present? && message_params[:send_by].present? && message_params[:receive_by].present? && message_params[:content].present? && message_params[:send_at].present?
        if current_user.id == (message_params[:send_by]).to_i || current_user.id == (message_params[:receive_by]).to_i
          @message = Message.new
          @message.content = message_params[:content]
          @message.send_at = message_params[:send_at]
          @message.user_send_by = User.find(message_params[:send_by])
          @message.user_receive_by = User.find(message_params[:receive_by])
          @message.save
          json_response(@message)
        end
      end
    end

    # GET messages/{id}/read/
    def read
      if current_user.id.present? && params[:id].present?
        @message = Message.find(params[:id])
        if current_user.id == @message.receive_by
          @message.read_at = DateTime.now
          @message.save
          json_response(@message)
       end
     end
    end

    # GET messages/search/{send_by, receive_by}
    def search
      if current_user.id.present? && message_params[:send_by].present? && message_params[:receive_by].present?
        if current_user.id == (message_params[:send_by]).to_i || current_user.id == (message_params[:receive_by]).to_i
          @message = Message.order('send_at DESC').where("send_by = ? AND receive_by = ?", message_params[:send_by], message_params[:receive_by])
          json_response(@message)
        end
      end
    end


    #GET messages/count/{receive_by}
    def count
      # retur integer
      # byebug
      if current_user.id.present? && message_params[:receive_by].present? && current_user.id == (message_params[:receive_by]).to_i
        total_messages = Message.where("read_at IS NULL AND receive_by = ?", message_params[:receive_by]).count
        @total_messages = {'total_messages' => total_messages}
        json_response(@total_messages)
      end
    end


    #List of hashes with users and number of messages received [{user_id:1,total_count:3}, {user_id:3, total_count:4}]
    #GET messages/count_send_by
    def count_send_by
      # byebug
      #USE THIS QUERY TO CREATE HASH FOR THE REQUEST
      # Message.where("read_at IS NULL").group(:send_by).pluck("send_by, count(*)")
      if current_user.id.present?
        @message = Message.where("read_at IS NULL and receive_by = ?", current_user.id).group(:send_by).pluck("send_by, count(*)").map { |p| {user_id: p[0], total_count: p[1]} }
        json_response(@message)
      end
    end

    

    private

    def message_params
      params.permit(:id, :content, :send_at, :send_by, :receive_by)
    end

  end
end
